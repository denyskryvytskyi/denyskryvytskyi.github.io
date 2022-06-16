---
layout: post
title: "Event System [Game engine]"
author: "Denys Kryvytskyi"
categories: journal
tags: [gameengine]
image: event-system/preview.png
---

Event system is one of the engine's core subsystem since we need to transfer data between different subsystems.

I was thinking about making non-blocking event system with queue feature and flexibility of use.

<h2 align="center"> Brief overview </h2>

Let's see what are the key components of our event system and how they interact with each other.
<br>
<br>
Firstly, I need to say that we will use some logic of publisher-subscriber pattern, but without correspond classes, just their logic.

So, we have the next components:
- Publisher. In our case - any code that send an event.
- Manager (bus, pool or any other name that you like). Main job - queue events and dispatch them.
- Subscriber. Any class with subscription on event.
- Event. User-defined (Event interface realization) class, that can be processed by manager by uuid.

Let's try to visualize Event and Manager on the class diagram.
We will not display any "publisher" or "subscriber" classes, because they are only user-dependent.
<p align="center">
   <img src="../assets/img/event-system/class-diagram.png" alt="Event system class diagram">
</p>

Next step is to visualize Publisher-Manager-Subscriber interaction on sequence diagram.
<p align="center">
   <img src="../assets/img/event-system/sequence-diagram.png" alt="Event system sequence diagram">
</p>

<h2 align="center"> Implementation </h2>

<h3 align="center">Event</h3>
Let's start our implementation from the core messaging thing - event.

Our Event interface is pretty simple:
{% highlight cpp %}
class Event
{
public:
   virtual const std::string GetEventType() const = 0;

public:
   bool Handled = false;
};
{% endhighlight %}

The main point is that every custom Event should have unique identifier, which we will use later in our manager.

Let me to show you example of custom event:
{% highlight cpp %}
 class WindowResizeEvent : public Event
{
public:
   MouseMovedEvent(float x, float y) :
         MouseX(x),
         MouseY(y)
   {}

   const std::string GetEventType() const override { return "{7CCF9526-19A3-431E-B9CB-B6AA7C775469}" };

public:
   float MouseX { 0.0f };
   float MouseY { 0.0f };
};
{% endhighlight %}

You can make some improvements (if you want), like more convenient uuid definition or debug tostring function (you can find my code [here](https://github.com/denyskryvytskyi/ElvenEngine/blob/master/Engine/src/Events/Event.h))

<h3 align="center"> Event callback </h3>
Fine, now we need to make callback wrapper. It's a trick (template magic &#128512;) that I've made to have possibility to subscribe on different events type by callbacks with corresponding event type reference (instead of unsafe Event pointer):
{% highlight cpp %}
class EventCallbackWrapper
{
public:
   void exec(Event& e)
   {
      call(e);
   }

   virtual const char* getType() const = 0;

private:
   virtual void call(Event& e) = 0;
};

template<typename EventType>
using EventFunctionHandler = std::function<void(EventType& e)>;

template<typename EventType>
class EventCallbackWrapperT : public EventCallbackWrapper
{
public:
   EventCallbackWrapperT(EventFunctionHandler<EventType> callback)
      : functionHandler(callback)
      , functionType(functionHandler.target_type().name())
   {};

private:
   virtual void call(Event& e) override
   {
      if (e.GetEventType() == EventType::GetStaticEventType())
      {
         functionHandler(static_cast<EventType&>(e));
      }
   }

   virtual const char* getType() const override { return functionType; }

private:
   EventFunctionHandler<EventType> functionHandler;
   const char* functionType;
};
{% endhighlight %}

I think that the code above pretty understandable without in depth explanation. But if you stuck - don't panic, just leave a comment below and I'll try to help you.

<h3 align="center"> Event Manager </h3>
The next code block is our event manager class:

{% highlight cpp %}
class EventManager
{
public:
   void Shutdown();

   void Subscribe(const std::string eventId, EventCallbackWrapper* handler);
   void Unsubscribe(const std::string eventId, const char* handlerName);
   void TriggerEvent(Event* event);
   void QueueEvent(Event* event);
   void DispatchEvents();

private:
   std::vector<Event*> m_eventsQueue;
   std::unordered_map<std::string, std::vector<EventCallbackWrapper*>> m_subscribers;
};

extern EventManager gEventManager;
{% endhighlight %}

I've created global variable for our manager, but it has some disadvantages with initialization order and multhireading perspective. So, you are free to make it local variable (maybe in Application class or whatever).

Also I've made useful functions to interact with Event Manager from any engine/game subsystem:

{% highlight cpp %}
template<typename EventType>
static void Subscribe(const EventFunctionHandler<EventType>& callback)
{
   EventCallbackWrapper* handler = new EventCallbackWrapperT<EventType>(callback);

   gEventManager.Subscribe(EventType::GetStaticEventType(), handler);
}

template<typename EventType>
static void Unsubscribe(const EventFunctionHandler<EventType>& callback)
{
   const char* handlerName = callback.target_type().name();
   gEventManager.Unsubscribe(EventType::GetStaticEventType(), handlerName);
}

static void TriggerEvent(Event* triggeredEvent)
{
   gEventManager.TriggerEvent(triggeredEvent);
}

static void QueueEvent(Event* queuedEvent)
{
   gEventManager.QueueEvent(queuedEvent);
}
{% endhighlight %}

As you can see, we use two data structures:
- vector for events queue, which contains pointer to events with different types.
- unordered_map (hashtable implementation from standard library), where key is event uuid (will be explained in Event section) and value - vector of subscribers-callbacks.

Subscribe/Unsubscribe functions should be called from subscriber's code.
TriggerEvent/QueueEvent function should be called from pusblisher's code.

DispatchEvents should be called from the main game loop on every, like this:

{% highlight cpp %}
void Application::Run()
{
   ...
   while(isApplicationRun)
   {
      Render();
      ProcessInput();

      gEventManager.DispatchEvents();
   }
   ...
}
{% endhighlight %}

Definition of our manager mechanism is next:

{% highlight cpp %}
void EventManager::Shutdown()
{
   for (auto event : m_eventsQueue)
   {
      delete event;
   }

   for (auto& eventSubscirbersIt = m_subscribers.begin(); eventSubscirbersIt != m_subscribers.end(); ++eventSubscirbersIt)
   {
      auto& handlers = eventSubscirbersIt->second;
      for (auto& it = handlers.begin(); it != handlers.end();)
      {
         delete* it;
         it = handlers.erase(it);
      }
   }
   m_subscribers.clear();
}

void EventManager::Subscribe(const std::string eventId, EventCallbackWrapper* handler)
{
   auto subscribers = m_subscribers.find(eventId);
   if (subscribers != m_subscribers.end())
   {
      auto& handlers = subscribers->second;
      for (auto it = handlers.begin(); it != handlers.end(); ++it)
      {
         if ((*it)->getType() == handler->getType())
         {
            EL_ASSERT(false, "Attempting to double-register callback");
            return;
         }
      }
      handlers.emplace_back(handler);
   }
   else
   {
      m_subscribers[eventId].emplace_back(handler);
   }
}

void EventManager::Unsubscribe(const std::string eventId, const char* handlerName)
{
   auto& handlers = m_subscribers[eventId];
   for (auto it = handlers.begin(); it != handlers.end(); ++it)
   {
      if ((*it)->getType() == handlerName)
      {
         delete* it;
         it = handlers.erase(it);
         return;
      }
   }
}

void EventManager::TriggerEvent(Event* event)
{
   for (auto handler : m_subscribers[event->GetEventType()])
   {
      handler->exec(*event);
   }
   event->Handled = true;
}

void EventManager::QueueEvent(Event* event)
{
   m_eventsQueue.emplace_back(event);
}

void EventManager::DispatchEvents()
{
   for (auto eventIt = m_eventsQueue.begin(); eventIt != m_eventsQueue.end();)
   {
      if (!(*eventIt)->Handled)
      {
         TriggerEvent(*eventIt);
         eventIt = m_eventsQueue.erase(eventIt);
      }
      else
      {
         ++eventIt;
      }
   }
}
{% endhighlight %}

If this code formatting is painful for you, you are free to look at the same [in-engine code on Github](https://github.com/denyskryvytskyi/ElvenEngine/blob/master/Engine/src/Events/EventManager.cpp).

<h3 align="center">Handler</h3>

Let's see how handler can interact with event manager.

For example, we have camera class, that create projection matrix.
Projection matrix depennds on window size. So, we need to handle window resize event to recalculate projection matrix every time we change window size.

Firstly, we add member function `OnWindowResizeEvent`, that will be our callback for event.

{% highlight cpp %}
class Camera
{
public:
   Camera()
   ~Camera()
   ... camera class code ...

   void OnWindowResizeEvent(const WindowResizeEvent& e);
};
{% endhighlight %}

Our ctor/dtor and callback definition:

{% highlight cpp %}
Camera::Camera()
{
   auto callback = std::bind(&Camera::OnWindowResized, this, std::placeholders::_1)
   Subscribe<WindowResizeEvent>(callback);
}

Camera::~Camera()
{
   auto callback = std::bind(&Camera::OnWindowResized, this, std::placeholders::_1)
   Unsubscribe<Events::WindowResizeEvent>(m_windowResizeCallback);
}

void Camera::OnWindowResized(const WindowResizeEvent& e)
{
   const int width = e.Width;
   const int height = e.Height;

   ... recalculate projection matrix by new window size and aspect ratio ...
}
{% endhighlight %}

To make it more simple, we can add member callback variable for camera class and bind it once in constructor. Also we can make macros as std::bind wrapper and send only member function as argument:

{% highlight cpp %}
#define EVENT_CALLBACK(fn) std::bind(&fn, this, std::placeholders::_1)

class Camera
{
public:
   Camera()
   ~Camera()
   ... camera class code ...

   void OnWindowResized(const WindowResizeEvent& e);

public:
   EventFunctionHandler<WindowResizeEvent> m_windowResizeCallback;
};

Camera::Camera()
{
   m_windowResizeCallback = EVENT_CALLBACK(Camera::OnWindowResized);
   Subscribe<WindowResizeEvent>(m_windowResizeCallback);
}

Camera::~Camera()
{
   Unsubscribe<Events::WindowResizeEvent>(m_windowResizeCallback);
}

void Camera::OnWindowResized(const WindowResizeEvent& e)
{
   const int width = e.Width;
   const int height = e.Height;

   ... recalculate projection matrix by new window size and aspect ratio ...
}
{% endhighlight %}

<h3 align="center"> Publisher</h3>

The last component of out system - publisher.
Actually our publisher is just send event call TriggerEvent/QueueEvent.

For example, we want send window resize event from window class:

{% highlight cpp %}
class Window
{
   ... window class code ...

   void Update
   {
      ... code ...

      setWindowResizeCallback([](int width, int height){
         TriggerEvent(new WindowResizeEvent(width, height));
      })
   }
}
{% endhighlight %}

<h3 align="center"> P.S. </h3>

I hope you will find something interesting and useful from this post.
I think I'll add support multithreading to event system when I'll experiment with multithreading in engine and I'll write post about it later.
Feel free to comment and write yor thoughts about my implementation.

Thank you for attention!

### References

- Game Coding Complete, 4th ed. *Mike McShaffry, David Graham*.
- Game Programming Patterns. *Robert Nystrom*.
- Game Engine Architecture, 3rd ed. *Jason Gregory*.
