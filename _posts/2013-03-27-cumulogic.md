---
layout: post
authors: [bart_blommaerts]
title: "MongoDb on HP Cloud, using Cumulogic"
tags: [mongodb,hp cumulogic]
category: Java
comments: true
---

On <a title="Cumulogic" href="http://www.cumulogic.com/" target="_blank">Cumulogic</a>'s website it is stated that Cumulogic is an enabler for rapidly building <a title="AWS" href="http://aws.amazon.com/" target="_blank">Amazon-style cloud services</a> securely on a private cloud and virtualized environment. Cumulogic is a platform that offers a <a title="PaaS" href="http://en.wikipedia.org/wiki/Platform_as_a_service" target="_blank">PaaS</a> (supporting Spring), a <a title="DBaaS" href="http://en.wikipedia.org/wiki/Cloud_database" target="_blank">DBaaS</a> (supporting MongoDb) and other offerings for application development. The Cumulogic platform sits on top of an <a title="IaaS" href="http://en.wikipedia.org/wiki/Cloud_computing#Infrastructure_as_a_service_.28IaaS.29" target="_blank">IaaS</a> environment such as <a title="HPCS" href="https://www.hpcloud.com/" target="_blank">HP Cloud Services</a> (HPCS). Currently Cumulogic and HPCS offer <a title="Cumulogic on HPCS" href="http://hpcloud.cumulogic.com/cl/" target="_blank">a free 90 day trial</a>.

<p style="text-align: center;">  
  <img class="image fit" alt="cumulogic" src="/img/older/hpcs.jpg">
</p>

Although that's a lot of buzzwords, I was curious about <a title="MongoDb" href=" http://www.mongodb.org/" target="_blank">MongoDB</a> and Cumulogic seemed to offer a quick way of learning something about it.
<h1>Cumulogic</h1>
After registering, a dashboard was greeting me. On the dashboard, I could create a MongoDB instance through a simple user interface. Since I wanted to use the MongoDB instance from Java, the Access Details tab was particularly interesting. It contained the connection details, I needed later.

<p style="text-align: center;">  
  <img class="image fit" alt="cumulogic" src="/img/older/mongo1.jpg">
</p>

Next up was the application container .. Cumulogic links the application container/server (Tomcat, JBoss, GlassFish, ..) with the preferred framework (Java EE, Spring, ..). While this makes sense (sort of), I don't see this link as a necessity. I choose Spring and ended up with Tomcat.

<p style="text-align: center;">  
  <img class="image fit" alt="cumulogic" src="/img/older/tomcat.jpg">
</p>

Both are using HPCS as Target Cloud. The configuration of both the database and the framework/application container really only took a couple of clicks. So far their statement regarding "enabling AWS-style cloud services" was correct.

The Framework instance has an Action(s) column that allows you to deploy/undeploy/redeploy/delete/... applications. Unfortunately there is no way to find logfiles when the deploy failed. I contacted Cumulogic and they stated logfiles would become available soon. Accessing logfiles really is essential for application development. I wasn't able to SSH into my instances either. The Access Groups displayed port 22 as available, but I couldn't find an account anywhere. They also didn't provide me with one, when I asked about it. Another issue I encountered was missing information in their user interface, eg. there is no way to check the JDK version on the instance, let alone change it. As it turned out, my Tomcat instance was using JDK 6, while I was trying to deploy a JDK 7 application.
<h1>MongoDB and Spring</h1>
I wanted to make an application that had CRUD functionality using Spring. I used Maven for building the application. In my Maven POM, I started with these dependencies:

<pre>
        &lt;dependency&gt;
            &lt;groupId&gt;org.springframework&lt;/groupId&gt;
            &lt;artifactId&gt;spring-core&lt;/artifactId&gt;
            &lt;version&gt;3.2.2.RELEASE&lt;/version&gt;
        &lt;/dependency&gt;

        &lt;dependency&gt;
            &lt;groupId&gt;org.springframework&lt;/groupId&gt;
            &lt;artifactId&gt;spring-context&lt;/artifactId&gt;
            &lt;version&gt;3.2.2.RELEASE&lt;/version&gt;
        &lt;/dependency&gt;

        &lt;dependency&gt;
            &lt;groupId&gt;org.mongodb&lt;/groupId&gt;
            &lt;artifactId&gt;mongo-java-driver&lt;/artifactId&gt;
            &lt;version&gt;2.10.1&lt;/version&gt;
        &lt;/dependency&gt;

        &lt;dependency&gt;
            &lt;groupId&gt;org.springframework.data&lt;/groupId&gt;
            &lt;artifactId&gt;spring-data-mongodb&lt;/artifactId&gt;
            &lt;version&gt;1.1.1.RELEASE&lt;/version&gt;
        &lt;/dependency&gt;

        &lt;dependency&gt;
            &lt;groupId&gt;javax.servlet&lt;/groupId&gt;
            &lt;artifactId&gt;servlet-api&lt;/artifactId&gt;
            &lt;version&gt;2.5&lt;/version&gt;
            &lt;scope&gt;provided&lt;/scope&gt;
        &lt;/dependency&gt;
</pre>

The servlet-api dependency is not necessary for using Spring and MongoDB, but I want to deploy the application on the Cumulogic Tomcat container and will use a simple servlet for that.

<a title="Spring Data" href="http://www.springsource.org/taxonomy/term/29" target="_blank">Spring Data</a> has a lot of support for MongoDB. I went with <a title="AbstractMongoConfiguration" href="http://static.springsource.org/spring-data/data-document/docs/1.0.0.BUILD-SNAPSHOT/spring-data-mongodb/apidocs/org/springframework/data/document/mongodb/config/AbstractMongoConfiguration.html" target="_blank">AbstractMongoConfiguration</a> for configuring MongoDB in Spring:

<pre>
public class SpringMongoConfig extends AbstractMongoConfiguration {

    @Override
    protected String getDatabaseName() {
        return &quot;mdb_test&quot;;
    }

    @Override
    @Bean
    public Mongo mongo() throws Exception {
        return new Mongo(&quot;15.185.160.1&quot;, 27017);
    }
</pre>

I needed something to persist and cars are always a good idea. I created this simple POJO:

<pre>
    public Car(String brand, String model, Integer horsePower) {
        this.brand = brand;
        this.model = model;
        this.horsePower = horsePower;
    }
</pre>

I'm pretty sure anyone can imagine what the rest of the class looks like, but I'll add the source of the application at the end. After a database configuration and an object to persist, I added a service class to interact with the database. The interesting part of this service is the <a title="MongoOperations" href="http://static.springsource.org/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/MongoOperations.html" target="_blank">MongoOperations</a> class. The API documentation holds all the necessary information for different operations such as insert, remove, update, ... CarService uses <a href="http://static.springsource.org/spring-data/mongodb/docs/current/api/org/springframework/data/mongodb/core/MongoTemplate.html" title="MongoTemplate" target="_blank">MongoTemplate</a>. Some might remember this from HibernateTemplate or JdbcTemplate.

<pre>
public class CarService {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(
            SpringMongoConfig.class);

    MongoOperations mongoOperation = (MongoOperations) ctx
            .getBean(&quot;mongoTemplate&quot;);

    public List&lt;Car&gt; findCars() {
        return mongoOperation.findAll(Car.class, &quot;cars&quot;);
    }

    public void save(Car car) {
        mongoOperation.save(car, &quot;cars&quot;);
    }
}
</pre>

All that's still needed now, is a Servlet to use the CarService class:

<pre>
/**
 * This Servlet was quickly hacked for demonstrating MongoDb.&lt;br /&gt;
 * Do not use it for real work. Ever.
 */
public class MongoServlet extends HttpServlet {

    private static final long serialVersionUID = -3353123142320733735L;

    public void doGet(HttpServletRequest req, HttpServletResponse res)
            throws ServletException, IOException {
        PrintWriter out = res.getWriter();

        CarService carService = new CarService();

        Car newCar = makeCar(req);
        if (newCar != null) {
            carService.save(newCar);
        }

        for (Car car : carService.findCars()) {
            out.println(car.getBrand() + &quot; - &quot; + car.getModel() + &quot; - &quot; + car.getHorsePower());
        }

        out.close();
    }

    private Car makeCar(HttpServletRequest req) {
        String[] brand = req.getParameterValues(&quot;brand&quot;);

        if (brand != null &amp;amp;&amp;amp; brand.length == 1) {
            String[] model = req.getParameterValues(&quot;model&quot;);
            String[] horsePower = req.getParameterValues(&quot;horsepower&quot;);

            return new Car(brand[0], model[0], Integer.parseInt(horsePower[0]));
        } else {
            return null;
        }
    }
</pre>

Please read the Javadoc on the class. Never use a servlet like this for a "real" project. It can go wrong in so many ways .. I consider it a quick and dirty hack to use my CarService. For it to work, a simple web.xml is needed:

<pre>
    &lt;servlet&gt;
        &lt;servlet-name&gt;MongDb&lt;/servlet-name&gt;
        &lt;servlet-class&gt;com.hp.afterhours.mongo.web.MongoServlet&lt;/servlet-class&gt;
    &lt;/servlet&gt;

    &lt;servlet-mapping&gt;
        &lt;servlet-name&gt;MongDb&lt;/servlet-name&gt;
        &lt;url-pattern&gt;/mongo/*&lt;/url-pattern&gt;
    &lt;/servlet-mapping&gt;
</pre>

And we're done.

Deploying the application shouldn't pose any problems. After the deployment, the content of the database can be queried using the servlet: (where localhost is replaced with the IP of your Tomcat instance)
<pre>http://localhost:1234/mongo</pre>
&nbsp;
Adding a new car to the MongoDB, can be done via
<pre>http://localhost:1234/mongo?brand=Mercedes&amp;model=SLS&amp;horsepower=583</pre>
&nbsp;
I implemented a couple methods of MongoOperations more in the CarService, in the attached source.

This simple example only scratches the surface of what's possible in MongoDB. <a title="Parleys" href="http://www.parleys.com/" target="_blank">Parleys</a> hosts an interesting talk on <a title="MongoDB" href="http://parleys.com/#st=5&amp;id=2219&amp;sl=1" target="_blank">"Building Web Applications with MongoDB"</a> (it has <a title="MongoDB" href="http://parleys.com/#st=5&amp;id=2220" target="_blank">2 parts</a>) that is worth checking if you want to learn more about MongoDB.

Source: 
<pre>
https://github.com/bart-blommaerts/cumulogic-mongo.git
</pre>