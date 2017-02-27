## 可运行应用 ##


## 应用结构 ##


## 注解启动方式 ##
使用注解方式启动应用，详细内容可以查看笔记：

## WebAppConfig ##
```JAVA
package com.spr.init;

import java.util.Properties;

import javax.annotation.Resource;
import javax.sql.DataSource;

import org.hibernate.ejb.HibernatePersistence;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.context.support.ResourceBundleMessageSource;
import org.springframework.core.env.Environment;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.view.InternalResourceViewResolver;
import org.springframework.web.servlet.view.JstlView;
import org.springframework.web.servlet.view.UrlBasedViewResolver;

import com.mchange.v2.c3p0.ComboPooledDataSource;

@Configuration
@EnableWebMvc
@EnableTransactionManagement
@ComponentScan("com.spr")
@PropertySource(value = { "classpath:application.properties", "classpath:c3p0.properties" })
@EnableJpaRepositories("com.spr.repository")
public class WebAppConfig {
    // c3p0
    private static final String PROPERTY_NAME_C300_DRIVER = "driverClass";
    private static final String PROPERTY_NAME_C300_PASSWORD = "password";
    private static final String PROPERTY_NAME_C300_URL = "jdbcUrl";
    private static final String PROPERTY_NAME_C300_USER = "user";
    private static final String PROPERTY_NAME_C3P0_MINIPOOLSIZE = "miniPoolSize";
    private static final String PROPERTY_NAME_C3P0_MAXPOOLSIZE = "maxPoolSize";
    private static final String PROPERTY_NAME_C3P0_INITIALPOOLSIZE = "initialPoolSize";
    private static final String PROPERTY_NAME_C3P0_MAXIDLETIME = "maxIdleTime";
    private static final String PROPERTY_NAME_C3P0_ACQUIREINCREMENT = "acquireIncrement";
    private static final String PROPERTY_NAME_C3P0_ACQUIRERETRYATTEMPTS = "acquireRetryAttempts";
    private static final String PROPERTY_NAME_C3P0_ACQUIRERETRYDELAY = "acquireRetryDelay";
    private static final String PROPERTY_NAME_C3P0_TESTCONNECTIONONCHECKIN = "testConnectionOnCheckin";
    private static final String PROPERTY_NAME_C3P0_AUTOMATICTESTTABLE = "automaticTestTable";
    private static final String PROPERTY_NAME_C3P0_IDLECONNECTIONTESTPERIOD = "idleConnectionTestPeriod";
    private static final String PROPERTY_NAME_C3P0_CHECKOUTTIMEOUT = "checkoutTimeout";

    // messageSouce
    private static final String PROPERTY_NAME_MESSAGE_SOURCE_BASENAME = "message.source.basename";
    // hibernate
    private static final String PROPERTY_NAME_HIBERNATE_DIALECT = "hibernate.dialect";
    private static final String PROPERTY_NAME_HIBERNATE_SHOW_SQL = "hibernate.show_sql";
    private static final String PROPERTY_NAME_ENTITYMANAGER_PACKAGES_TO_SCAN = "entitymanager.packages.to.scan";
    private static final String PROPERTY_NAME_HIBERNATE_HMB2DDL_AUTO = "hibernate.hbm2ddl.auto";
    private static final String PROPERTY_NAME_HIBERANTE_FORMAT_SQL = "hiberante.format_sql";

    @Resource
    private Environment env;

    @Bean
    public DataSource dataSource() throws Exception {
        ComboPooledDataSource comboPooledDataSource = new ComboPooledDataSource();
        comboPooledDataSource.setDriverClass(env.getRequiredProperty(PROPERTY_NAME_C300_DRIVER));
        comboPooledDataSource.setUser(env.getRequiredProperty(PROPERTY_NAME_C300_USER));
        comboPooledDataSource.setJdbcUrl(env.getRequiredProperty(PROPERTY_NAME_C300_URL));
        comboPooledDataSource.setPassword(env.getRequiredProperty(PROPERTY_NAME_C300_PASSWORD));
        comboPooledDataSource.setAcquireIncrement(Integer.valueOf(env.getRequiredProperty(PROPERTY_NAME_C3P0_ACQUIREINCREMENT)));
        comboPooledDataSource.setMaxIdleTime(Integer.valueOf(env.getRequiredProperty(PROPERTY_NAME_C3P0_MAXIDLETIME)));
        comboPooledDataSource.setMinPoolSize(Integer.valueOf(env.getRequiredProperty(PROPERTY_NAME_C3P0_MINIPOOLSIZE)));
        comboPooledDataSource.setMaxPoolSize(Integer.valueOf(env.getRequiredProperty(PROPERTY_NAME_C3P0_MAXPOOLSIZE)));
        comboPooledDataSource.setInitialPoolSize(Integer.valueOf(env.getRequiredProperty(PROPERTY_NAME_C3P0_INITIALPOOLSIZE)));
        comboPooledDataSource.setAcquireRetryAttempts(Integer.valueOf(env.getRequiredProperty(PROPERTY_NAME_C3P0_ACQUIRERETRYATTEMPTS)));
        comboPooledDataSource.setAcquireRetryDelay(Integer.valueOf(env.getRequiredProperty(PROPERTY_NAME_C3P0_ACQUIRERETRYDELAY)));
        comboPooledDataSource.setTestConnectionOnCheckin(Boolean.valueOf(env.getRequiredProperty(PROPERTY_NAME_C3P0_TESTCONNECTIONONCHECKIN)));
        comboPooledDataSource.setAutomaticTestTable(env.getRequiredProperty(PROPERTY_NAME_C3P0_AUTOMATICTESTTABLE));
        comboPooledDataSource.setIdleConnectionTestPeriod(Integer.valueOf(env.getRequiredProperty(PROPERTY_NAME_C3P0_IDLECONNECTIONTESTPERIOD)));
        comboPooledDataSource.setCheckoutTimeout(Integer.valueOf(env.getRequiredProperty(PROPERTY_NAME_C3P0_CHECKOUTTIMEOUT)));
        return comboPooledDataSource;
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory() throws Exception {
        LocalContainerEntityManagerFactoryBean entityManagerFactoryBean = new LocalContainerEntityManagerFactoryBean();
        entityManagerFactoryBean.setDataSource(dataSource());
        entityManagerFactoryBean.setPersistenceProviderClass(HibernatePersistence.class);
        entityManagerFactoryBean.setPackagesToScan(env.getRequiredProperty(PROPERTY_NAME_ENTITYMANAGER_PACKAGES_TO_SCAN));

        entityManagerFactoryBean.setJpaProperties(hibProperties());

        return entityManagerFactoryBean;
    }

    private Properties hibProperties() {
        Properties properties = new Properties();
        properties.put(PROPERTY_NAME_HIBERNATE_DIALECT, env.getRequiredProperty(PROPERTY_NAME_HIBERNATE_DIALECT));
        properties.put(PROPERTY_NAME_HIBERNATE_SHOW_SQL, env.getRequiredProperty(PROPERTY_NAME_HIBERNATE_SHOW_SQL));
        properties.put(PROPERTY_NAME_HIBERNATE_HMB2DDL_AUTO,
                env.getRequiredProperty(PROPERTY_NAME_HIBERNATE_HMB2DDL_AUTO));
        properties.put(PROPERTY_NAME_HIBERANTE_FORMAT_SQL, env.getRequiredProperty(PROPERTY_NAME_HIBERANTE_FORMAT_SQL));
        return properties;
    }

    @Bean
    public JpaTransactionManager transactionManager() throws Exception {
        JpaTransactionManager transactionManager = new JpaTransactionManager();
        transactionManager.setEntityManagerFactory(entityManagerFactory().getObject());
        return transactionManager;
    }

    @Bean
    public UrlBasedViewResolver setupViewResolver() {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/pages/");
        resolver.setSuffix(".jsp");
        resolver.setViewClass(JstlView.class);
        return resolver;
    }

    @Bean
    public ResourceBundleMessageSource messageSource() {
        ResourceBundleMessageSource source = new ResourceBundleMessageSource();
        source.setBasename(env.getRequiredProperty(PROPERTY_NAME_MESSAGE_SOURCE_BASENAME));
        source.setUseCodeAsDefaultMessage(true);
        return source;
    }

}
```
- `dataSource()`使用c3p0连接池
- `entityManagerFactory()`配置orm为hibernate
- `transactionManager()`配置事务
- `messageSource()`i18n国际化
- `@EnableWebMvc`启用mvc注解
- `@EnableTransactionManagement`启用事务注解
- `@ComponentScan("com.spr")`扫描spring注解
- `@PropertySource(value = { "classpath:application.properties", "classpath:c3p0.properties" })`加载配置文件
- `@EnableJpaRepositories("com.spr.repository")`扫描jpa注解

## jpaRepository ##
```JAVA
package com.spr.repository;

import org.springframework.data.jpa.repository.JpaSpecificationExecutor;
import org.springframework.data.repository.CrudRepository;
import org.springframework.stereotype.Repository;

import com.spr.model.Shop;

@Repository("autowiringShopJpaRepository")
public interface AutowiringShopJpaRepository extends CrudRepository<Shop, Integer>, JpaSpecificationExecutor<Shop> {

}
```

```JAVA
package com.spr.repository;

import javax.annotation.Resource;

import org.springframework.dao.EmptyResultDataAccessException;
import org.springframework.stereotype.Repository;

import com.spr.model.Shop;

@Repository("shopRepository")
public class JpaShopRepository implements ShopRepository {
    @Resource(name = "autowiringShopJpaRepository")
    AutowiringShopJpaRepository autowiringShopJpaRepository;

    public Iterable<Shop> findAll() {
        return autowiringShopJpaRepository.findAll();
    }

    public Shop save(Shop shop) {
        autowiringShopJpaRepository.save(shop);
        return null;
    }

    public Shop findOne(int id) throws EmptyResultDataAccessException {
        Shop findShop = autowiringShopJpaRepository.findOne(id);
        if (findShop == null) {
            throw new EmptyResultDataAccessException(String.format("No %s entity with id %s exists!", Shop.class, id), 1);
        }
        return null;
    }

    public void delete(Shop deletedShop) {
        autowiringShopJpaRepository.delete(deletedShop);
    }

}
```
不直接继承`CrudRepository<Shop, Integer>`，而是使用`AutowiringShopJpaRepository`继承，之后在`JpaShopRepository`中操作。这样能够在`JpaShopRepository`类中做一些异常捕获和简单的处理。

## 数据校验 ##
```JAVA
package com.spr.validation;

import org.springframework.stereotype.Component;
import org.springframework.validation.Errors;
import org.springframework.validation.ValidationUtils;
import org.springframework.validation.Validator;

import com.spr.model.Shop;

@Component
public class ShopValidator implements Validator {
	
	private final static String EMPLOYEES_NUMBER = "emplNumber";

	@Override
	public boolean supports(Class<?> clazz) {
		return Shop.class.isAssignableFrom(clazz);
	}

	@Override
	public void validate(Object target, Errors errors) {
		Shop shop = (Shop) target;
		
		Integer emplNumber = shop.getEmplNumber();
		
		ValidationUtils.rejectIfEmpty(errors, "name", "shop.name.empty");
		ValidationUtils.rejectIfEmpty(errors, EMPLOYEES_NUMBER, "shop.emplNumber.empty");
		
		if (emplNumber != null && emplNumber < 1)
			errors.rejectValue(EMPLOYEES_NUMBER, "shop.emplNumber.lessThenOne");

	}

}
```
通过继承`Validator`方式实现，相比于直接使用`hibernate validator`注解麻烦一些。hibernate validator注解方式查阅笔记：【】【】【】

## ExceptionHandle ##
```JAVA
package com.spr.controller;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpEntity;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.ResponseStatus;

import com.spr.exception.ShopNotFound;
import com.spr.model.RestError;

@ControllerAdvice
public class ShopControllerAdvice {
    private static final Logger logger = LoggerFactory.getLogger(ShopControllerAdvice.class);

    @ExceptionHandler(value = { ShopNotFound.class })
    @ResponseStatus(value = HttpStatus.BAD_REQUEST)
    @ResponseBody
    public HttpEntity<RestError> handleShopNotFoundException(ShopNotFound exception) {
        return new ResponseEntity<RestError>(new RestError(exception.getErrorCode(), exception.getMessage()), HttpStatus.BAD_REQUEST);
    }
}
```
## controller ##
```JAVA
package com.spr.controller;

import java.util.List;

import javax.validation.Valid;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.InitBinder;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import com.spr.exception.ShopNotFound;
import com.spr.model.Shop;
import com.spr.service.ShopService;
import com.spr.validation.ShopValidator;

@Controller
@RequestMapping(value = "/shop")
public class ShopController {

    @Autowired
    private ShopService shopService;

    @Autowired
    private ShopValidator shopValidator;

    @InitBinder
    private void initBinder(WebDataBinder binder) {
        binder.setValidator(shopValidator);
    }

    @RequestMapping(value = "/create", method = RequestMethod.GET)
    public ModelAndView newShopPage() {
        ModelAndView mav = new ModelAndView("shop-new", "shop", new Shop());
        return mav;
    }

    @RequestMapping(value = "/create", method = RequestMethod.POST)
    public ModelAndView createNewShop(@ModelAttribute @Valid Shop shop,
            BindingResult result,
            final RedirectAttributes redirectAttributes) {

        if (result.hasErrors())
            return new ModelAndView("shop-new");

        ModelAndView mav = new ModelAndView();
        String message = "New shop " + shop.getName() + " was successfully created.";

        shopService.create(shop);
        mav.setViewName("redirect:/index.html");

        redirectAttributes.addFlashAttribute("message", message);
        return mav;
    }

    @RequestMapping(value = "/list", method = RequestMethod.GET)
    public ModelAndView shopListPage() {
        ModelAndView mav = new ModelAndView("shop-list");
        List<Shop> shopList = shopService.findAll();
        mav.addObject("shopList", shopList);
        return mav;
    }

    @RequestMapping(value = "/edit/{id}", method = RequestMethod.GET)
    public ModelAndView editShopPage(@PathVariable Integer id) {
        ModelAndView mav = new ModelAndView("shop-edit");
        Shop shop = shopService.findById(id);
        mav.addObject("shop", shop);
        return mav;
    }

    @RequestMapping(value = "/edit/{id}", method = RequestMethod.POST)
    public ModelAndView editShop(@ModelAttribute @Valid Shop shop,
            BindingResult result,
            @PathVariable Integer id,
            final RedirectAttributes redirectAttributes) throws ShopNotFound {

        if (result.hasErrors())
            return new ModelAndView("shop-edit");

        ModelAndView mav = new ModelAndView("redirect:/index.html");
        String message = "Shop was successfully updated.";

        shopService.update(shop);

        redirectAttributes.addFlashAttribute("message", message);
        return mav;
    }

    @RequestMapping(value = "/delete/{id}", method = RequestMethod.DELETE)
    public ModelAndView deleteShop(@PathVariable Integer id,
            final RedirectAttributes redirectAttributes) throws ShopNotFound {

        ModelAndView mav = new ModelAndView("redirect:/index.html");

        Shop shop = shopService.delete(id);
        String message = "The shop " + shop.getName() + " was successfully deleted.";

        redirectAttributes.addFlashAttribute("message", message);
        return mav;
    }

}
```
## pom.xml ##
可能pom文件才是最重要的
```XML
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

	<modelVersion>4.0.0</modelVersion>
	<groupId>nd.esp.com</groupId>
	<artifactId>SpringMvcJpaC3p0ExceptionHandle</artifactId>
	<packaging>war</packaging>
	<version>0.0.1-SNAPSHOT</version>
	<name>spr-data</name>
	
	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<spring.framework.version>4.2.3.RELEASE</spring.framework.version>
		<spring.version>4.2.3.RELEASE</spring.version>
		<hibernate.version>4.2.3.Final</hibernate.version>
		<hibernate.validator>5.2.4.Final</hibernate.validator>
		<logback.version>1.0.13</logback.version>
		<slf4j.version>1.7.5</slf4j.version>
		<spring.data.jpa.version>1.10.0.RELEASE</spring.data.jpa.version>
	</properties>
	
	<build>
		<finalName>dpr-data</finalName>
		<plugins>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>2.3.2</version>
				<configuration>
					<source>1.7</source>
					<target>1.7</target>
				</configuration>
			</plugin>
		</plugins>
	</build>

	<dependencies>
		<dependency>
			<groupId>c3p0</groupId>
			<artifactId>c3p0</artifactId>
			<version>0.9.1.2</version>
		</dependency>
		
		<!-- <dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-annotations</artifactId>
			<version>2.7.3</version>
		</dependency> -->
		<dependency>
		    <groupId>com.fasterxml.jackson.core</groupId>
		    <artifactId>jackson-databind</artifactId>
		    <version>2.5.0</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-tx</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-orm</artifactId>
			<version>${spring.version}</version>
		</dependency> 
		
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-core</artifactId>
			<version>${hibernate.version}</version>
		</dependency>
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-validator</artifactId>
			<version>${hibernate.validator}</version>
		</dependency>
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-entitymanager</artifactId>
			<version>${hibernate.version}</version>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.25</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${spring.version}</version>
		</dependency>
		
		<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-jpa</artifactId>
			<version>${spring.data.jpa.version}</version>
			<exclusions>
				<exclusion>
					<artifactId>spring-asm</artifactId>
					<groupId>org.springframework</groupId>
				</exclusion>
			</exclusions>
		</dependency>
		
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.0.1</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>jstl</groupId>
			<artifactId>jstl</artifactId>
			<version>1.2</version>
		</dependency>
        <!-- slf4j -->
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>${slf4j.version}</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>${slf4j.version}</version>
		</dependency>
		<!-- log4jdbc -->
		<!-- <dependency>
			<groupId>com.googlecode.log4jdbc</groupId>
			<artifactId>log4jdbc</artifactId>
			<version>1.2</version>
			<scope>runtime</scope>
		</dependency> -->
	</dependencies>
</project>
```
