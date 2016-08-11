Cloud Audit
===================

Infrastructure for logging request/response pairs and other debugging/auditing info.

First off logging into Google BigQuery.

Uses Spring.

## Features

### AuditItem model

Provides Immutables based model for AuditItems of type LOG, EXCEPTION, REQUEST & RESPONSE

### Audit To BigQuery

AuditItemTable is a Big Query table definition which should be extended to add your required tracking attributes.

You should subclass with something like:

    @Component
    public class MyAuditor extends AuditItemTable {
    
        @Autowired
        public MyAuditor(AuditTableIdentifier identifier) {
            super(identifier);
        }
    
        @Override
        protected Map<String, TableFieldSchema> customFields() {
            Map<String,TableFieldSchema> fields = new LinkedHashMap<>();
            TableFieldSchema tracking = field("tracking", FieldType.RECORD, FieldMode.NULLABLE);
            tracking.setFields(new ArrayList<>());
            tracking.getFields().add(field("internal_tracing_id",FieldType.STRING,FieldMode.NULLABLE));
            tracking.getFields().add(field("logical_session_id",FieldType.STRING,FieldMode.NULLABLE));
            fields.put("tracking",tracking);    
            return fields;
        }
    }

### DefaultAuditFilter

When configured to run as a servlet filter this will map request headers into a TrackingMap for use within AuditItem.
It will audit the incoming request and outgoing response to the configured auditor.

You will need to subclass DefaultAuditFilter like:

    @Component
    public class MyAuditFilter extends DefaultAuditFilter {
    
        @Autowired
        public MyAuditFilter(BackgroundAuditor<AuditItem> auditor) {
            super(auditor);
        }
    }

And then configure with something like:

    @Configuration
    @ComponentScan(basePackages = {"com.cloudburst"})
    public class MyAuditFilterConfiguration {
    
        @Bean
        @Autowired
        public FilterRegistrationBean auditFilterRegistration(MyAuditFilter filter) {
            FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
            filterRegistrationBean.setFilter(filter);
            filterRegistrationBean.setOrder(1); // ordering in the filter chain
            return filterRegistrationBean;
        }
    
    }

### Logback Appender

Add this to your logback.xml to WARN and ERROR to audit DB:

    <appender name="AUDIT" class="com.cloudburst.audit.logback.AuditAppender"/>
    
If your TrackingMap contains something like audit-level=DEBUG then you will get DEBUG and INFO as well. 
This means that you can potentially add request headers in order to elevate the audit level for that request.

### JAX-WS Handler

Add new AuditSOAPHandler() to your SOAP handler chain in order to audit the SOAP requests and responses.

## To Release new version to Bintray

    mvn clean release:prepare -Darguments="-Dmaven.javadoc.skip=true"
    mvn release:perform -Darguments="-Dmaven.javadoc.skip=true"


