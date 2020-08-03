Below is the behavior of the SDK on receiving an exception from the AMQP library being used underneath. If the exception is marked as retryable, the exception will get translated to an `AmqpIoTResourceException` internally, and the SDK will implement the default retry-policy and attempt to reconnect. For exceptions not marked as retryable, it is advised to inspect the exception details and perform the necessary action as indicated below.

* NOTE - The SDK default [retry policy](./retrypolicy.md) is `ExponentialBackoff`. 

|Exception Name |Details/Inner exception |Error code (if available) |isRetryable  |Action                 |
|------|------|------|------|------|
| | | | 
| AmqpIoTResourceException | SDK will retry, this exception will not bubble up to the application directly | amqp:connection:forced error | Yes | SDK will retry |
| AmqpIoTResourceException | SDK will retry, this exception will not bubble up to the application directly | amqp:connection:framing-error | Yes | SDK will retry |
| AmqpIoTResourceException | SDK will retry, this exception will not bubble up to the application directly | amqp:connection:redirect | Yes | SDK will retry |
| IotHubThrottledException | Always generated from an AmqpOutcome <br/> SDK will retry, this exception will not bubble up to the application directly | com.microsoft:device-container-throttled | Yes | SDK will retry <br/>TODO: if SDK retries, then is IotHubThrottledException ever thrown to the application?? |
| IotHubCommunicationException | InnerException: AmqpIoTResourceException | amqp:decode-error | No | Mismatch between AMQP message sent by client and received by service; collect logs and contact service <br/>TODO:  AmqpIoTResourceException has isTransient = false, but IotHubCommunicationException has isTransient = true - will SDK retry??|
| IotHubCommunicationException | InnerException: AmqpIoTResourceException | amqp:frame-size-too-small | No | The AMQP message is not being formed correctly by the SDK, collect logs and contact SDK team <br/>TODO:  AmqpIoTResourceException has isTransient = false, but IotHubCommunicationException has isTransient = true - will SDK retry??|
| IotHubException | InnerException: AmqpException.Error.Condition = AmqpSymbol.IllegalState | amqp:illegal-state | No | Inspect the exception details, collect logs and contact service <br/> TODO: when does service team throw this??|
| IotHubCommunicationException | InnerException: AmqpException.Error.Condition = AmqpSymbol.InternalError <br/> SDK will retry, this exception will not bubble up to the application directly | amqp:internal-error | Yes | SDK will retry |
| InvalidOperationException | InnerException: AmqpException.Error.Condition = AmqpSymbol.InvalidField | amqp:invalid-field | No | Inspect the exception details, collect logs and contact service |
| AmqpIoTResourceException | SDK will retry, this exception will not bubble up to the application directly | amqp:link :detach-forced | Yes | SDK will retry |
| MessageTooLargeException | InnerException: AmqpException.Error.Condition = AmqpSymbol.MessageSizeExceeded | amqp:link :message-size-exceeded | No | The AMQP message size exceeded the value supported by the link, collect logs and contact service |
| AmqpIoTResourceException	| SDK will retry, this exception will not bubble up to the application directly | amqp:link :redirect | Yes | SDK will retry | 
| AmqpIoTResourceException | SDK will retry, this exception will not bubble up to the application directly | amqp:link stolen | Yes | SDK will retry |
| AmqpIoTResourceException | SDK will retry, this exception will not bubble up to the application directly | amqp:link :transfer-limit-exceeded | Yes | SDK will retry |
| InvalidOperationException	| InnerException: AmqpException.Error.Condition = AmqpSymbol.NotAllowed | amqp:not-allowed | No | Inspect the exception details, collect logs and contact service |
| DeviceNotFoundException | This is the exception thrown when the error is received as an AmqpException | amqp:not-found | No | Inspect the exception details, collect logs and contact service |
| DeviceMessageLockLostException | This is the exception thrown when the error is received as an AmqpOutcome | amqp:not-found | No | Inspect the exception details, collect logs and contact service |
| NotSupportedException | InnerException: AmqpException.Error.Condition = AmqpSymbol.NotImplemented | amqp:not-implemented | No | Inspect the exception details, collect logs and contact service |
| IotHubException | InnerException: AmqpException.Error.Condition = AmqpSymbol.PreconditionFailed | amqp:precondition-failed | No | Inspect the exception details, collect logs and contact service |
| IotHubException | InnerException: AmqpException.Error.Condition = AmqpSymbol.ResourceDeleted | amqp:resource-deleted | No | Inspect the exception details, collect logs and contact service |
| IotHubException | InnerException: AmqpException.Error.Condition = AmqpSymbol.ResourceLimitExceeded  amqp:resource-limit-exceeded | No | Inspect the exception details, collect logs and contact service |
| AmqpIoTResourceException | SDK will retry, this exception will not bubble up to the application directly | amqp:resource-locked | Yes | SDK will retry |
| AmqpIoTResourceException | SDK will retry, this exception will not bubble up to the application directly | amqp:session:errant-link | Yes | SDK will retry |
| AmqpIoTResourceException | SDK will retry, this exception will not bubble up to the application directly | amqp:session:handle-in-use | Yes | SDK will retry |
| AmqpIoTResourceException | SDK will retry, this exception will not bubble up to the application directly | amqp:session:unattached-handle | Yes | SDK will retry |
| AmqpIoTResourceException | SDK will retry, this exception will not bubble up to the application directly | amqp:session:window-violation | Yes | SDK will retry |
| UnauthorizedException | InnerException: AmqpException.Error.Condition = AmqpSymbol.UnauthorizedAccess | amqp:unauthorized-access | No | Inspect your credentials |

* NOTE - For exceptions marked as retryable, the SDK will implement its retry policy internally, and you do not need to take any action. For non-retryable exceptions, or after the SDK retry policy has expired, you should inspect both the connection status change handler and the returned exception details to determine the next step.

* If the device is in `Connected` state, you can perform subsequent operations on the same client instance.
* If the device is in `Disconnected_Retrying` state, then the SDK is retrying to recover its connection. Wait until device recovers and reports a `Connected` state, and then perform subsequent operations.
* If the device is in `Disconnected` or `Disabled` state, then the underlying transport layer has been disposed. You should dispose of the existing `DeviceClient` instance and then initialize a new client (initializing a new `DeviceClient` instance without disposing the previously used instance will cause them to fight for the same connection resources).