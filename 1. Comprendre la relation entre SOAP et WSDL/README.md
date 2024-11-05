# Comprendre la relation entre SOAP et WSDL

## 1. Qu’est-ce que SOAP ?

SOAP (Simple Object Access Protocol) est un protocole de communication standard qui permet à des applications d'échanger des informations sur un réseau. SOAP est souvent utilisé dans les entreprises parce qu'il est robuste et fonctionne bien dans des environnements où des applications doivent communiquer de manière fiable et sécurisée, même si elles utilisent des langages et plateformes différents.

SOAP repose sur XML pour formater les messages, qui sont souvent envoyés sur HTTP ou HTTPS. Un message SOAP est structuré en trois parties principales :

- **Envelope** : La partie principale qui encapsule tout le message SOAP.
- **Header** (facultatif) : Contient des informations de routage ou de sécurité.
- **Body** : Contient les données principales de la requête ou de la réponse.

### Exemple de message SOAP

Voici un exemple de message SOAP demandant la température d’une ville spécifique :

```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Body>
        <getTemperature xmlns="http://example.com/weather">
            <city>Paris</city>
        </getTemperature>
    </soap:Body>
</soap:Envelope>
```

Dans cet exemple :
- Le message demande la température pour "Paris".
- `getTemperature` est la méthode appelée dans le service web.

> **Rappel** : SOAP définit **comment** envoyer les messages, en utilisant XML pour structurer l’information, mais il ne dit rien sur **ce que le service offre comme fonctionnalités**.

## 2. Qu’est-ce que WSDL ?

WSDL (Web Services Description Language) est un document XML qui **décrit** les services web SOAP, en fournissant les informations suivantes :

- **Quelles méthodes sont disponibles** : Par exemple, des méthodes comme `getTemperature`, `getForecast`.
- **Quels paramètres chaque méthode attend** : Par exemple, `city` est un paramètre pour `getTemperature`.
- **Quels types de données sont utilisés** : Par exemple, `city` est une chaîne de caractères (string).
- **Où le service peut être appelé** : l’URL du service.

Le fichier WSDL permet donc à un client de savoir **ce que le service peut faire** et **comment formater les requêtes** pour communiquer avec lui.

### Exemple de WSDL

Voici un extrait simplifié d’un fichier WSDL pour un service météo :

```xml
<definitions xmlns="http://schemas.xmlsoap.org/wsdl/"
             xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/"
             targetNamespace="http://example.com/weather">

    <!-- Déclaration des types de données -->
    <types>
        <schema xmlns="http://www.w3.org/2001/XMLSchema" targetNamespace="http://example.com/weather">
            <element name="getTemperatureRequest">
                <complexType>
                    <sequence>
                        <element name="city" type="string"/>
                    </sequence>
                </complexType>
            </element>
        </schema>
    </types>

    <!-- Messages échangés -->
    <message name="getTemperatureRequest">
        <part name="parameters" element="tns:getTemperatureRequest"/>
    </message>

    <!-- Opérations disponibles -->
    <portType name="WeatherServicePortType">
        <operation name="getTemperature">
            <input message="tns:getTemperatureRequest"/>
            <output message="tns:getTemperatureResponse"/>
        </operation>
    </portType>

    <!-- Liaison avec SOAP -->
    <binding name="WeatherServiceBinding" type="tns:WeatherServicePortType">
        <soap:binding transport="http://schemas.xmlsoap.org/soap/http" style="document"/>
        <operation name="getTemperature">
            <soap:operation soapAction="getTemperature"/>
            <input>
                <soap:body use="literal"/>
            </input>
            <output>
                <soap:body use="literal"/>
            </output>
        </operation>
    </binding>

    <!-- URL du service -->
    <service name="WeatherService">
        <port name="WeatherServicePort" binding="tns:WeatherServiceBinding">
            <soap:address location="http://localhost:8080/weather"/>
        </port>
    </service>
</definitions>
```

### Explication

Le fichier WSDL décrit les fonctionnalités offertes par le service (les méthodes disponibles, les paramètres attendus) et comment le contacter.

## 3. Relation entre SOAP et WSDL

La relation entre SOAP et WSDL peut être comprise ainsi :

- **SOAP** définit le format des messages (comme `Envelope`, `Header`, `Body`) et comment ils sont transmis.
- **WSDL** décrit les fonctionnalités offertes par le service (les méthodes disponibles, les paramètres attendus) et comment le contacter.

Lorsqu’un client souhaite utiliser un service web SOAP, il consulte le fichier WSDL pour savoir **comment formater les requêtes** et **comment utiliser le service**.

### Exemple de l’utilisation de SOAP et WSDL ensemble

1. Le client lit le fichier WSDL du service pour comprendre les opérations disponibles.
2. Il voit que `getTemperature` prend un paramètre `city` de type `string`.
3. En utilisant ces informations, le client construit un message SOAP.

### Exemple Complet en Java

Voici comment créer un client en Java pour appeler un service SOAP en utilisant le WSDL.

#### Étape 1 : Générer le Code Client avec le WSDL

```bash
wsimport -keep -verbose http://localhost:8080/weather?wsdl
```

#### Étape 2 : Utiliser le Code Généré

```java
import javax.xml.namespace.QName;
import javax.xml.ws.Service;
import java.net.URL;

public class WeatherClient {
    public static void main(String[] args) throws Exception {
        URL url = new URL("http://localhost:8080/weather?wsdl");
        QName qname = new QName("http://example.com/weather", "WeatherService");

        Service service = Service.create(url, qname);
        WeatherService weather = service.getPort(WeatherService.class);

        String temperature = weather.getTemperature("Paris");
        System.out.println("Temperature in Paris: " + temperature);
    }
}
```

---

## Conclusion

SOAP et WSDL sont des éléments essentiels pour les services web :
- **SOAP** est le protocole de communication qui permet d’envoyer et de recevoir des messages XML.
- **WSDL** est la description du service, indiquant les opérations disponibles et les données attendues.

Ils permettent de créer des applications capables de communiquer de manière fiable, interopérable et sécurisée.
