# Bugreport

When running this project it will succeed, however it shows logging that contains a warning.
My goal it to have a warningless build (and even tests without any logging), but this one cannot be removed. 
It is also not clear what the issue is and how to solve it. 

```
[INFO] Running com.sourcegrounds.lesslogging.ApplicationIT
19:56:25,401 |-WARN in Logger[org.springframework.core.env.PropertySourcesPropertyResolver] - No appenders present in context [default] for logger [org.springframework.core.env.PropertySourcesPropertyResolver].
19:56:25,424 |-INFO in ch.qos.logback.classic.model.processor.RootLoggerModelHandler - Setting level of ROOT logger to OFF
19:56:25,425 |-INFO in ch.qos.logback.classic.jul.LevelChangePropagator@680362a - Propagating OFF level on Logger[ROOT] onto the JUL framework
19:56:25,427 |-INFO in ch.qos.logback.core.model.processor.DefaultProcessor@3569edd5 - End of configuration.
19:56:25,427 |-INFO in org.springframework.boot.logging.logback.SpringBootJoranConfigurator@1f651cd8 - Registering current configuration as safe fallback point

[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 3.84 s - in com.sourcegrounds.lesslogging.ApplicationIT
```

It seems the root cause is in the LoggingSystemProperties in case one of these logging-properties is set, because the PropertyResolver wants to log it when a key is found.

```
	protected void apply(LogFile logFile, PropertyResolver resolver) {
		setSystemProperty(resolver, EXCEPTION_CONVERSION_WORD, "logging.exception-conversion-word");
		setSystemProperty(PID_KEY, new ApplicationPid().toString());
		setSystemProperty(resolver, CONSOLE_LOG_PATTERN, "logging.pattern.console");
		setSystemProperty(resolver, CONSOLE_LOG_CHARSET, "logging.charset.console", getDefaultCharset().name());
		setSystemProperty(resolver, LOG_DATEFORMAT_PATTERN, "logging.pattern.dateformat");
		setSystemProperty(resolver, FILE_LOG_PATTERN, "logging.pattern.file");
		setSystemProperty(resolver, FILE_LOG_CHARSET, "logging.charset.file", getDefaultCharset().name());
		setSystemProperty(resolver, LOG_LEVEL_PATTERN, "logging.pattern.level");
		if (logFile != null) {
			logFile.applyToSystemProperties();
		}
	}
```

The bug was discovered when working with spring-cloud-stream-test-support [org.springframework.cloud.stream.binder.BinderTestEnvironmentPostProcessor](https://github.com/spring-cloud/spring-cloud-stream/blob/main/core/spring-cloud-stream-test-support/src/main/java/org/springframework/cloud/stream/binder/BinderTestEnvironmentPostProcessor.java#L39-L40) and transformed in this minimal testcase.
