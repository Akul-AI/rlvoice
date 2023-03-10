Implementing drivers
--------------------

You can implement new drivers for the :mod:`rlvoice.Engine` by:

#. Creating a Python module with the name of your new driver.
#. Implementing the required driver factory function and class in your module.
#. Using methods on a :class:`rlvoice.driver.DriverProxy` instance provided by the :class:`rlvoice.Engine` to control the event queue and notify applications about events.

The Driver interface
~~~~~~~~~~~~~~~~~~~~

All drivers must implement the following factory function and driver interface.

.. module:: rlvoice.drivers
   :synopsis: The package containing the available driver implementations

.. function:: buildDriver(proxy : rlvoice.driver.DriverProxy) -> rlvoice.drivers.DriverDelegate

   Instantiates delegate subclass declared in this module.
   
   :param proxy: Proxy instance provided by a :class:`rlvoice.Engine` instance.

.. class:: DriverDelegate
   
   .. note:: The :class:`DriverDelegate` class is not actually declared in :mod:`rlvoice.drivers` and cannot serve as a base class. It is only here for the purpose of documenting the interface all drivers must implement.

   .. method:: __init__(proxy : rlvoice.drivers.DriverProxy, *args, **kwargs) -> None

      Constructor. Must store the proxy reference.
      
      :param proxy: Proxy instance provided by the :func:`buildDriver` function.

   .. method:: destroy() ->
   
      Optional. Invoked by the :class:`rlvoice.driver.DriverProxy` when it is being destroyed so this delegate can clean up any synthesizer resources. If not implemented, the proxy proceeds safely.

   .. method:: endLoop() -> None
   
      Immediately ends a running driver event loop.
   
   .. method:: getProperty(name : string) -> object
   
      Immediately gets the named property value. At least those properties listed in the :meth:`rlvoice.Engine.getProperty` documentation must be supported.
      
      :param name: Name of the property to query.
      :return: Value of the property at the time of this invocation.
   
   .. method:: say(text : unicode, name : string) -> None
   
      Immediately speaks an utterance. The speech must be output according to the current property values applied at the time of this invocation. Before this method returns, it must invoke :meth:`rlvoice.driver.DriverProxy.setBusy` with value :const:`True` to stall further processing of the command queue until the output completes or is interrupted.
      
      This method must trigger one and only one `started-utterance` notification when output begins, one `started-word` notification at the start of each word in the utterance, and a `finished-utterance` notification when output completes.
      
      :param text: Text to speak.
      :param name: Name to associate with the utterance. Included in notifications about this utterance.
   
   .. method:: setProperty(name : string, value : object) -> None
   
      Immediately sets the named property value. At least those properties listed in the :meth:`rlvoice.Engine.setProperty` documentation must be supported. After setting the property, the driver must invoke :meth:`rlvoice.driver.DriverProxy.setBusy` with value :const:`False` to pump the command queue.
      
      :param name: Name of the property to change.
      :param value: Value to set.
   
   .. method:: startLoop()
   
      Immediately starts an event loop. The loop is responsible for sending notifications about utterances and pumping the command queue by using methods on the :class:`rlvoice.driver.DriverProxy` object given to the factory function that created this object.
   
   .. method:: stop()
   
      Immediately stops the current utterance output. This method must trigger a `finished-utterance` notification if called during on-going output. It must trigger no notification if there is no ongoing output. 
      
      After stopping the output and sending any required notification, the driver must invoke :meth:`rlvoice.driver.DriverProxy.setBusy` with value :const:`False` to pump the command queue.

The DriverProxy interface
~~~~~~~~~~~~~~~~~~~~~~~~~

.. module:: rlvoice.driver
   :synopsis: The module containing the driver proxy implementation

The :func:`rlvoice.drivers.buildDriver` factory receives an instance of a :class:`DriverProxy` class and provides it to the :class:`rlvoice.drivers.DriverDelegate` it constructs. The driver delegate can invoke the following public methods on the proxy instance. All other public methods found in the code are reserved for use by an :class:`rlvoice.Engine` instance.

.. class:: DriverProxy
   
   .. method:: isBusy() -> bool
   
      Gets if the proxy is busy and cannot process the next command in the queue or not.
   
      :return: True means busy, False means idle.

   .. method:: notify(topic : string, **kwargs) -> None
   
      Fires a notification.
      
      :param topic: The name of the notification.
      :kwargs: Name/value pairs associated with the topic.
   
   .. method:: setBusy(busy : bool) -> None
   
      Sets the proxy to busy so it cannot continue to pump the command queue or idle so it can process the next command.
      
      :param busy: True to set busy, false to set idle
