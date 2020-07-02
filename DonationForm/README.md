# Stripe API as a Moqui Component
This document will guide you through the process of creating a component in Moqui, using [Stripe's API](https://stripe.com/) to [collect customer payment information](https://stripe.com/docs/checkout) as well as [creating charges](https://stripe.com/docs/charges) to their credit card. 

## Clone the Moqui Repo
First, clone or copy the Moqui Framework [here](https://github.com/moqui/moqui-framework). Once the framework has been cloned to your machine, cd into it and run the following ./gradlew commands (click [here](https://docs.gradle.org/current/userguide/gradle_wrapper.html#gradle_wrapper) for more information on Gradle Wrapper) to set up your Moqui framework :
- `$ ./gradlew getComponent -Pcomponent=HiveMind`
- `$ ./gradlew addRuntime`
- `$ ./gradlew getComponent -Pcomponent=example`
- `$ ./gradlew getComponent -Pcomponent=PopCommerce `

When finished, you can run your project by running `$./gradlew build load run` and going to http://localhost:8080. Once you have confirmed that your web app is running, open up the project in any text editor.

## Creating the screens
1. Create a new directory in the `/runtime/components` folder with the name of your component. In this example, we will create a component called **DonationPage**.
2. Create the following things inside `../DonationPage`
    - a `build` folder,
    - a `data` folder,
    - an `entity` folder,
    - a `screen` folder,
    - a `script` folder,
    - a `service` folder,
    - a `src` folder,
    - a `build.gradle` file,
    - a `component.xml` file,
    - and a `MoquiConf.xml` file.

## Creating your first screen
1. Inside the `../DonationPage/screen` folder, create the following items:
    - a `DonationPage.xml` file,
    - a `DonationPage` folder
2. Inside the `DonationPage.xml` file, add the following code:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<screen require-authentication="false" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/xml-screen-2.1.xsd">
    <subscreens default-item="StripeForm.html"/>
    <widgets>
        <subscreens-active/>
    </widgets>
</screen>
``` 

This code is responsible for the screen that our app goes to when going to `http://localhost:8080/apps/DonationPage`.
3. As you may have noticed, our DonationPage screen is rendering a subscreen in the <widgets> element. Above this, we have a *default-item* defined for our subcreen that is rendered. This *default-item* is calling a subscreen named `StripeForm.html`. Let's create that subsceen now. Inside `../DonationPage/screen/DonationPage`, create a file named `StripeForm.html` and copy the following code into it: 
```html
<form action="your-server-side-code" method="POST">
    <script
      src="https://checkout.stripe.com/checkout.js" class="stripe-button"
      data-key="pk_test_Jigk57YQcA1vMohz4lE6vDcd"
      data-amount="999"
      data-name="Stripe.com"
      data-description="Widget"
      data-image="https://stripe.com/img/documentation/checkout/marketplace.png"
      data-locale="auto"
      data-zip-code="true">
    </script>
  </form>
  ```
  This is the code from [Stripe API Docs](https://stripe.com/docs/checkout) that creates a button on the webpage that looks like this:
  ![alt text][screenshot1]
  
  When this button is clicked, it will render a modal that has a form that a customer can fill out with his/her credit card information. 
  ![alt text][screenshot2]
  
  When the user clicks on the submit button, the information is sent to Stripe for processing (read more about it [here](https://stripe.com/docs/checkout)). In the meantime, we want the user to be redirected to some kind of confirmation page. We will create that next.
  4. Inside `../DonationPage/screen/DonationPage`, create a file named `ThankYouPage.xml` and copy the following code into it: 
  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
<screen require-authentication="false" standalone="true"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/xml-screen-2.1.xsd">
    
    <transition name="Stripe" require-session-token="false">
        <service-call name="DonationPage.DonationPageServices.createDonation"/>
        <default-response url="."/>
    </transition>

    <widgets>
        <label type="h1" text="Thank you for your generous donation!"/>
    </widgets>
</screen>
```
This subscreen should be loaded when the user has just submitted a payment. This subscreen has a transition that we have named "Stripe". The purpose of this transition is to perform a service-call that actually creates a charge on the user's credit card as well as redirect the user to this Thank You Page. Let's go back to `../DonationPage/screen/DonationPage/StripeForm.html` and change the action attribute of the form from `"your-server-side-code"` to `"DonationPage/ThankYouPage/Stripe"`. The code for `../DonationPage/screen/DonationPage/StripeForm.html` should now look like this:
```html
<form action="DonationPage/ThankYouPage/Stripe" method="POST">
    <script
      src="https://checkout.stripe.com/checkout.js" class="stripe-button"
      data-key="YOUR TEST PUBLIC API KEY"
      data-amount="999"
      data-name="Stripe.com"
      data-description="Widget"
      data-image="https://stripe.com/img/documentation/checkout/marketplace.png"
      data-locale="auto"
      data-zip-code="true">
    </script>
  </form>
  ```
  **Remember to change the value of `data-key` to whatever your test public API key is. (You can find this key in your Stripe account.)**
  
  
  What this does is when the form with the user's information is submitted, the service "Stripe" in ``../DonationPage/screen/DonationPage/ThankYouPage.xml` is triggered by the URL http://localhost:8080/apps/DonationPage/ThankYouPage/Stripe. Immediately, the Stripe service is run which calls another service that we will later define as well as redirect to http://localhost:8080/apps/DonationPage/ThankYouPage (due to the default-response attribute) which shows the thank you message.
  
  The `require-session-token="false"` in the <transition> element is necessary to grant the user permission to access the transition.
  
  ## Some Configurations
  At this point, we will do some configuration. In your `MoquiConf.xml` file, paste the following code into it:
  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
<!-- No copyright or license for configuration file, details here are not considered a creative work. -->
<moqui-conf
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/moqui-conf-2.1.xsd">
    <screen-facade>
        <screen location="component://webroot/screen/webroot/apps.xml">
            <subscreens-item name="DonationPage" menu-title="Demo Application" menu-index="2" location="component://DonationPage/screen/DonationPage.xml" menu-include="Y"/>
        </screen>
    </screen-facade>
</moqui-conf>
```
Make sure that the "name" and "location" attributes in the <subscreens-item/> element reflect the name of your component. This configuration file will tell Moqui to load the screens we just created when we access the component we made in the browser.

In `component.xml`, add this block of code:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<component xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/moqui-conf-2.1.xsd"
        name="DonationPage" version="1.3.0"/>
```
The most important thing is to make sure that the value of "name" is set to equal the name of the component, which in this example is "DonationPage".

## Building the Stripe Services
Now, let's create the services that actually charge the user after their credit card information has been received by Stripe. Create a file named `DonationPageServices.xml` at this directory: `../component/DonationPage/service/DonationPage`. In the file, add this code: 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<services xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://moqui.org/xsd/service-definition-2.1.xsd">
    <service verb="create" noun="Donation" type="java" location="com.mk.moqui.Stripe" method="getDonation" authenticate="anonymous-all">
    </service>
</services>
```
This code creates the service (`DonationPage.DonationPageServices.createDonation`) that is called when the user executes the transition named "Stripe" in the thank you page.

Now, you'll notice that the service here doesn't perform any actions itself, but calls another service located at "com.mk.moqui.Stripe". This new service is a Groovy service that we're calling. The reason we are created a Groovy service is because Stripe already has boiler-plate code written in Java (which Groovy can interpret) that can create the charges.

Now, create the following folder structure: `../src/main/groovy/com/mk/moqui`. Inside `../com/mk/moqui`, create a file named `Stripe.groovy` and add the following code:
```groovy
package com.mk.moqui;
import org.moqui.context.ExecutionContext;
import java.util.HashMap;
import java.util.Map;
import com.stripe.Stripe;
import com.stripe.exception.StripeException;
import com.stripe.model.Charge;
import com.stripe.net.RequestOptions;

class Stripe {
    public static void getDonation(ExecutionContext ec) {
        Stripe.apiKey = "YOUR TEST SECRET API KEY";
        //  String token = request.getParameter("stripeToken");
        Map<String, Object> params = new HashMap<String, Object>();

        params.put("amount", 999);
        params.put("currency", "usd");
        params.put("source", "tok_visa");
        params.put("receipt_email", "jenny.rosen@example.com");
        Charge charge = Charge.create(params);
    }
}
```
This source code can be found out [here](https://stripe.com/docs/charges). This is the code that will actually charge the user the amount he/she decided to pay. 

**Remember to change the value of `Stripe.apiKey` to whatever your test secret API key is. (You can find this key in your Stripe account.)**

Finally, to make sure Moqui can run all the Stripe-related services, you will need to configure your `build.gradle` file to have the necessary dependency. Make sure your `build.gradle` file has the following code: 

```gradle
apply plugin: 'groovy'

sourceCompatibility = '1.8'

def jarBaseName = 'CompiledComponentName' // Customize compiled file name
def componentNode = parseComponent(project)
def moquiDir = file(projectDir.absolutePath + '/../../..')
def frameworkDir = file(moquiDir.absolutePath + '/framework')

version = componentNode.'@version'

repositories {
    flatDir name: 'localLib', dirs: frameworkDir.absolutePath + '/lib'
    jcenter()
}

tasks.withType(JavaCompile) { options.compilerArgs << "-proc:none" }
tasks.withType(GroovyCompile) { options.compilerArgs << "-proc:none" }

dependencies {
    compile project(':framework')
    testCompile project(':framework').configurations.testCompile.allDependencies
    compile "com.stripe:stripe-java:7.1.0"
}

task cleanLib(type: Delete) { delete fileTree(dir: projectDir.absolutePath+'/lib', include: '*') }
clean.dependsOn cleanLib

jar {
    destinationDir = file(projectDir.absolutePath + '/lib') // Declaring compiled Groovy goes into lib directory
    baseName = jarBaseName
}
task copyDependencies { doLast {
    copy { from (configurations.runtime - project(':framework').configurations.runtime - project(':framework').jar.archivePath)
        into file(projectDir.absolutePath + '/lib') }
} }
copyDependencies.dependsOn cleanLib
jar.dependsOn copyDependencies
```

**Now, we should be good to go. Since we made updates to the MoquiConf.xml, run `$ ./gradlew load run` and go to `http://localhost:8080/apps/DonationPage` to view your component!