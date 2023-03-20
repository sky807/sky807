### Spring Boot Logback 설정

---

- Logback이란?

  - Java에서 Log 기록을 위한 오픈소스 SLF4J의 구현체 
  - Spring Boot에서는 별도의 라이브러리 추가X

- Spring Boot Logback 설정 파일

  Spring이나 일반 Java 프로그램의 경우 logback.xml 파일을 resources 디렉터리 만들어서 참조하지만

  Spring Boot에서는 아래 3가지중 한 가지 방법 사용 

  - application.properties에 설정
  - resources/logback-spring.xml에 설정 (이 방법 선택)
  - resources/logback.xml에 설정 

- Log level 순서 및 사용 방법 

  TRACE < DEBUG < INFO < WARN < ERROR

  - ERROR : 요청을 처리하는 중 오류가 발생한 경우 
  - WARN : 처리 가능한 문제, 향수 시스템 에러의 원인이 될 수 있는 경고성 메세지 표시
  - INFO : 상태 변경과 같은 정보성 로그를 표시 
  - DEBUG : 프로그램 디버깅 하기 위한 정보표시 
  - TRACE : 추적 레벨은 DEBUG보다 훨씬 상세한 정보표시

- logback-spring.xml 설정

  > [참고] logback.properties 설정

  > spring boot에서는 간단하게 application.properties에서도 log 설정을 할 수 있지만 상세한 설정을 위해서는 logback-spring.xml에 설정을 따로 해야 합니다.

  > 그런데, xml은 복잡하기 때문에 파일 경로나 파일명 등은 수정하기 쉽도록 별도로 properties 파일로 분리하기도 합니다.

  ```
    log.config.path=c:/logs/
    log.config.filename=webapi-global.log
  ```

  ​

  ```
  <?xml version="1.0" encoding="UTF-8"?>
  <!-- 60초마다 설정 파일의 변경을 확인 하여 변경시 갱신 -->
  <configuration scan="true" scanPeriod="60 seconds">
    
    <!-- 프로퍼티 연결 -->
    <property resource="logback.properties"/>
      
    <!--Environment 내의 프로퍼티들을 개별적으로 설정할 수도 있다.-->
    <springProperty scope="context" name="LOG_LEVEL" source="logging.level.root"/>
    <!-- log file path -->
    <property name="LOG_PATH" value="${log.config.path}"/>
    <!-- log file name -->
    <property name="LOG_FILE_NAME" value="${log.config.filename}"/>
    <!-- err log file name -->
    <property name="ERR_LOG_FILE_NAME" value="err_log"/>
    <!-- pattern -->
    <property name="LOG_PATTERN" value="%-5level %d{yy-MM-dd HH:mm:ss}[%thread] [%logger{0}:%line] - %msg%n"/>
    
    
    <!-- Console Appender -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
      <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
        <pattern>${LOG_PATTERN}</pattern>
      </encoder>
    </appender>
        
    <!-- File Appender -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <!-- 파일경로 설정 -->
      <file>${LOG_PATH}/${LOG_FILE_NAME}.log</file>
      <!-- 출력패턴 설정-->
      <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
        <pattern>${LOG_PATTERN}</pattern>
      </encoder>
      
      <!-- Rolling 정책 -->
      <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <!-- .gz,.zip 등을 넣으면 자동 일자별 로그파일 압축 -->
        <fileNamePattern>${LOG_PATH}/global/${LOG_FILE_NAME}.%d{yyyy-MM-dd}_%i.log</fileNamePattern>
        <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
          <!-- 파일당 최고 용량 kb, mb, gb -->
          <maxFileSize>10MB</maxFileSize>
        </timeBasedFileNamingAndTriggeringPolicy>
        <!-- 일자별 로그파일 최대 보관주기(~일), 해당 설정일 이상된 파일은 자동으로 제거-->
        <maxHistory>30</maxHistory>
      </rollingPolicy>
    </appender>
    
    <!-- 에러의 경우 파일에 로그 처리 -->
    <appender name="ERROR_LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
      <filter class="ch.qos.logback.classic.filter.LevelFilter">
        <level>ERROR</level>
        <onMatch>ACCEPT</onMatch>
        <onMismatch>DENY</onMismatch>
      </filter>
      <file>${LOG_PATH}/${ERR_LOG_FILE_NAME}.log</file>
      <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
        <!-- 이 옵션이 없을 경우 한글이 깨지는 경우 있음-->
        <charset>UTF-8</charset>
        <pattern>${LOG_PATTERN}</pattern>
      </encoder>
      
      <!-- Rolling 정책 -->
      <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <!-- .gz,.zip 등을 넣으면 자동 일자별 로그파일 압축 -->
        <fileNamePattern>${LOG_PATH}/${ERR_LOG_FILE_NAME}.%d{yyyy-MM-dd}_%i.log</fileNamePattern>
        <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
          <!-- 파일당 최고 용량 kb, mb, gb -->
          <maxFileSize>10MB</maxFileSize>
        </timeBasedFileNamingAndTriggeringPolicy>
        <!-- 일자별 로그파일 최대 보관주기(~일), 해당 설정일 이상된 파일은 자동으로 제거-->
        <maxHistory>60</maxHistory>
      </rollingPolicy>
    </appender>
    
    <!-- 로그 형태 설정 -->
    <appender name="ONLINE" class="클래스 위치 (kr.co.test.FileLogConfigure)">
    	<path>${FILE_LOG_PATH}</path>
    	<fileName>online</fileName>
    	<filter class="ch.qos.logback.core.filter.EvaluatorFilter">
    		<evaluator class="ch.qos.lobback.classic.boolex.OnMarkerEvaluator">
    			<!-- log marker 설정 -->
    			<marker>ONLINE</marker>
    			<marker>TEST</marker>
    				...더 추가 가능
    		</evaluator>
    		<OnMatch>ACCEPT</OnMatch>
    		<OnMissmatch>DENY</OnMissmatch>
    	</filter>
    	</appender>
    
    <!-- 특정패키지 로깅레벨 설정 -->
    <springProfile name="dev, local">
      <logger name="org.apache.ibatis" level="INFO"/>
          <root level="${LOG_LEVEL}">
            <appender-ref ref="CONSOLE"/>
            <appender-ref ref="FILE"/>
            <appender-ref ref="Error"/>
            <appender-ref ref="ONLINE"/>
          </root>
     </springProfile>
     
     <springProfile name="qa, live">
      <logger name="org.apache.ibatis" level="INFO"/>
          <root level="${LOG_LEVEL}">
            <appender-ref ref="CONSOLE"/>
            <appender-ref ref="FILE"/>
            <appender-ref ref="Error"/>
            <appender-ref ref="ONLINE"/>
          </root>
     </springProfile>
        

  </configuration>
  ```

- FileLogConfigure Class 설정

  ```
  @Slf4j
  public class FileLogConfigure extends APpenderBase<ILoggingEvent> {
    
    private String path;
    
    private String fileName;
    
    public void setPath(String path) {
      this.path = path;
    }
    
    public void setFileName(String fileName) {
      this.fileName = fileName;
    }
    
    @Override
    protected void append(ILoggingEvent event) {
      String fileName = DateUtil.getDateString() + "-" + this.fileName;
      String dateTile = DateUtil.getDateTimeString(DateUtil.F_YYYYMMDDHHMMSS);
      
      File dir = new File(path);
      File file = new File(path + "\\" + fileName + ".log");
      
      if (!dir.exists()) dir.mkdirs();
      
      try {
        if (!file.exists()) file.createNewFile();
      } catch (IOException e) {
        log.error(e.toString());
      }
      
      try (PrintWriter pw = new PrintWriter(new BufferedWriter(new FileWriterWithEncoding(file, "UTF-8", true)))) {
        pw.write("[" + dateTime + "]" + "[" + event.getMarker() + "]" + event.toString() + "\r\n");
        pw.flush();
      } catch (IOException e) {
        log.error(e.toString());
      }
    }
  }
  ```

- LogMarker Class 설정

  ```
  public class LOGMARKER {
    public static ONLINE = MarkerFactory.getMarker("ONLINE");
    public static TEST = MarkerFactory.getMarker("TEST");
  	... 더 추가 가능
  }
  ```