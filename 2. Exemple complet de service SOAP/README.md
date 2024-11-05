# Exemple complet de service SOAP avec Spring Boot

Dans cet exemple, nous allons créer un service SOAP en Java avec Spring Boot pour fournir des informations sur un pays. Le service `CountryService` renverra la capitale et la population d'un pays donné.

### Étape 1 : Initialiser le projet Spring Boot

1. Créez un nouveau projet Spring Boot, par exemple avec [Spring Initializr](https://start.spring.io/).
2. Sélectionnez les dépendances suivantes :
   - **Spring Web Services** : Pour le support SOAP.
   - **Spring Boot DevTools** (facultatif) : Pour faciliter le développement.
   - **Spring Boot Starter Test** : Pour les tests unitaires.

### Étape 2 : Ajouter les dépendances dans `pom.xml`

Dans le fichier `pom.xml`, assurez-vous d’avoir les dépendances suivantes pour gérer les services SOAP et le mappage XML en Java avec JAXB.

```xml
<dependencies>
    <!-- Spring Boot Starter Web Services pour SOAP -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web-services</artifactId>
    </dependency>

    <!-- JAXB pour les objets Java générés à partir de XML -->
    <dependency>
        <groupId>javax.xml.bind</groupId>
        <artifactId>jaxb-api</artifactId>
    </dependency>

    <!-- Spring Boot Starter pour les tests -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>

Le fichier WSDL décrit les opérations que le service SOAP propose, les types de données utilisés, et où le service peut être contacté.

### Création du fichier `countries.wsdl`

Dans le dossier `src/main/resources`, créez un fichier `countries.wsdl` qui définit le service `CountryService`.

```xml
<definitions xmlns="http://schemas.xmlsoap.org/wsdl/"
             xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/"
             xmlns:tns="http://example.com/countries"
             xmlns:xsd="http://www.w3.org/2001/XMLSchema"
             targetNamespace="http://example.com/countries">
             
    <!-- Types de données utilisés par le service -->
    <types>
        <schema xmlns="http://www.w3.org/2001/XMLSchema" targetNamespace="http://example.com/countries">
            <element name="getCountryRequest">
                <complexType>
                    <sequence>
                        <element name="name" type="xsd:string"/>
                    </sequence>
                </complexType>
            </element>
            <element name="getCountryResponse">
                <complexType>
                    <sequence>
                        <element name="country" type="tns:country"/>
                    </sequence>
                </complexType>
            </element>
            <complexType name="country">
                <sequence>
                    <element name="name" type="xsd:string"/>
                    <element name="capital" type="xsd:string"/>
                    <element name="population" type="xsd:int"/>
                </sequence>
            </complexType>
        </schema>
    </types>

    <!-- Messages de la requête et réponse -->
    <message name="getCountryRequest">
        <part name="parameters" element="tns:getCountryRequest"/>
    </message>
    <message name="getCountryResponse">
        <part name="parameters" element="tns:getCountryResponse"/>
    </message>

    <!-- Opérations du service -->
    <portType name="CountryServicePortType">
        <operation name="getCountry">
            <input message="tns:getCountryRequest"/>
            <output message="tns:getCountryResponse"/>
        </operation>
    </portType>

    <!-- Liaison du service avec SOAP -->
    <binding name="CountryServiceSoapBinding" type="tns:CountryServicePortType">
        <soap:binding transport="http://schemas.xmlsoap.org/soap/http" style="document"/>
        <operation name="getCountry">
            <soap:operation soapAction="getCountry"/>
            <input>
                <soap:body use="literal"/>
            </input>
            <output>
                <soap:body use="literal"/>
            </output>
        </operation>
    </binding>

    <!-- Service et son endpoint -->
    <service name="CountryService">
        <port name="CountryServicePort" binding="tns:CountryServiceSoapBinding">
            <soap:address location="http://localhost:8080/ws"/>
        </port>
    </service>
</definitions>
```

#### Explication des sections

- **Types** : Déclare les structures pour la requête et la réponse, ainsi que le type de données `country`.
- **Messages** : Définit les messages échangés, `getCountryRequest` et `getCountryResponse`.
- **PortType** : Définit l’opération `getCountry`.
- **Binding** : Associe le `PortType` à SOAP.
- **Service** : Définit l’URL où le service sera disponible.

### Configuration de Spring Boot pour SOAP

Ajoutez une classe de configuration `WebServiceConfig.java` pour enregistrer le servlet SOAP et exposer le fichier WSDL.

#### Fichier `WebServiceConfig.java`

```java
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.ws.config.annotation.EnableWs;
import org.springframework.ws.transport.http.MessageDispatcherServlet;
import org.springframework.xml.xsd.SimpleXsdSchema;
import org.springframework.xml.xsd.XsdSchema;

@EnableWs
@Configuration
public class WebServiceConfig {

    @Bean
    public ServletRegistrationBean<MessageDispatcherServlet> messageDispatcherServlet(ApplicationContext context) {
        MessageDispatcherServlet servlet = new MessageDispatcherServlet();
        servlet.setApplicationContext(context);
        servlet.setTransformWsdlLocations(true);
        return new ServletRegistrationBean<>(servlet, "/ws/*");
    }

    @Bean(name = "countries")
    public XsdSchema countriesSchema() {
        return new SimpleXsdSchema(new ClassPathResource("countries.xsd"));
    }
}
```

### Création des classes Java pour le service

#### Classe `Country`

```java
public class Country {
    private String name;
    private String capital;
    private int population;

    // Getters et setters
}
```

#### Classe `CountryRepository`

Cette classe stocke des informations sur les pays.

```java
import java.util.HashMap;
import java.util.Map;

public class CountryRepository {
    private static final Map<String, Country> countries = new HashMap<>();

    public CountryRepository() {
        Country france = new Country("France", "Paris", 67000000);
        Country usa = new Country("United States", "Washington D.C.", 331000000);
        countries.put(france.getName(), france);
        countries.put(usa.getName(), usa);
    }

    public Country findCountry(String name) {
        return countries.get(name);
    }
}
```

### Définir l'endpoint du service SOAP

Créez la classe `CountryEndpoint` pour définir le point de terminaison du service SOAP, en utilisant les annotations `@Endpoint` et `@PayloadRoot`.

#### Classe `CountryEndpoint.java`

```java
import org.springframework.ws.server.endpoint.annotation.Endpoint;
import org.springframework.ws.server.endpoint.annotation.PayloadRoot;
import org.springframework.ws.server.endpoint.annotation.RequestPayload;
import org.springframework.ws.server.endpoint.annotation.ResponsePayload;

@Endpoint
public class CountryEndpoint {
    private static final String NAMESPACE_URI = "http://example.com/countries";
    private final CountryRepository countryRepository;

    public CountryEndpoint(CountryRepository countryRepository) {
        this.countryRepository = countryRepository;
    }

    @PayloadRoot(namespace = NAMESPACE_URI, localPart = "getCountryRequest")
    @ResponsePayload
    public GetCountryResponse getCountry(@RequestPayload GetCountryRequest request) {
        GetCountryResponse response = new GetCountryResponse();
        response.setCountry(countryRepository.findCountry(request.getName()));
        return response;
    }
}
```

### Tester le service SOAP

Démarrez l’application Spring Boot. Utilisez un outil comme **Postman** ou **SOAP UI** pour envoyer cette requête à `http://localhost:8080/ws`.

#### Requête SOAP

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:tns="http://example.com/countries">
   <soapenv:Header/>
   <soapenv:Body>
      <tns:getCountryRequest>
         <tns:name>France</tns:name>
      </tns:getCountryRequest>
   </soapenv:Body>
</soapenv:Envelope>
```

### Réponse SOAP

Si le service fonctionne correctement, il devrait renvoyer une réponse similaire à celle-ci :

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/">
   <soapenv:Body>
      <tns:getCountryResponse xmlns:tns="http://example.com/countries">
         <tns:country>
            <tns:name>France</tns:name>
            <tns:capital>Paris</tns:capital>
            <tns:population>67000000</tns:population>
         </tns:country>
      </tns:getCountryResponse>
   </soapenv:Body>
</soapenv:Envelope>
```

Cela conclut le test du service SOAP, qui est maintenant opérationnel et prêt à être utilisé.
