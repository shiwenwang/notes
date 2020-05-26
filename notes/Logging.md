---
tags: [Python]
title: Logging
created: '2020-03-27T02:33:49.179Z'
modified: '2020-03-27T03:06:52.134Z'
---

# Logging

### 日志级别
| 级别 | 数值 |
| -- | -- |
| CRITICAL | 50|
| ERROR | 40|
| WARNING | 30|
| INFO | 20|
| DEBUG | 10|
| NOTSET | 0|

### 日志格式化

|属性名称|格式|描述|
|--|--|--|
|args|不需要格式化。|The tuple of arguments merged into msg to produce message, or a dict whose values are used for the merge (when there is only one argument, and it is a dictionary).|
|asctime|%(asctime)s|Human-readable time when the LogRecord was created. By default this is of the form '2003-07-08 16:49:45,896' (the numbers after the comma are millisecond portion of the time).|
|created|%(created)f|Time when the LogRecord was created (as returned by time.time()).|
|exc_info|不需要格式化。|Exception tuple (à la sys.exc_info) or, if no exception has occurred, None.|
|filename|%(filename)s|Filename portion of pathname.|
|funcName|%(funcName)s|Name of function containing the logging call.|
|levelname|%(levelname)s|Text logging level for the message ('DEBUG', 'INFO', 'WARNING', 'ERROR', 'CRITICAL').|
|levelno|%(levelno)s|Numeric logging level for the message (DEBUG, INFO, WARNING, ERROR, CRITICAL).|
|lineno|%(lineno)d|Source line number where the logging call was issued (if available).|
|message|%(message)s|The logged message, computed as msg % args. This is set when Formatter.format() is invoked.|
|module|%(module)s|模块 (filename 的名称部分)。|
|msecs|%(msecs)d|Millisecond portion of the time when the LogRecord was created.|
|msg|不需要格式化。|The format string passed in the original logging call. Merged with args to produce message, or an arbitrary object (see 使用任意对象作为消息).|
|name|%(name)s|Name of the logger used to log the call.|
|pathname|%(pathname)s|Full pathname of the source file where the logging call was issued (if available).|
|process|%(process)d|进程ID（如果可用）|
|processName|%(processName)s|进程名（如果可用）|
|relativeCreated|%(relativeCreated)d|Time in milliseconds when the LogRecord was created, relative to the time the logging module was loaded.|
|stack_info|不需要格式化。|Stack frame information (where available) from the bottom of the stack in the current thread, up to and including the stack frame of the logging call which resulted in the creation of this record.|
|thread|thread)d|线程ID（如果可用）|
|threadName|%(threadName)s|线程名（如果可用）|
