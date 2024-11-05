# Comprendre SOAP - Partie 1 : Les Bases de SOAP

SOAP (Simple Object Access Protocol) est un protocole de communication qui permet d'échanger des informations structurées entre des applications dans un environnement réseau, souvent utilisé dans le contexte de services web. SOAP est particulièrement populaire dans les systèmes d'entreprise pour son standardisé, ce qui assure une interopérabilité entre différentes plateformes et langages.

SOAP repose sur XML (Extensible Markup Language) pour formater ses messages et fonctionne en utilisant des protocoles réseau standard, principalement HTTP ou HTTPS. Voici les concepts clés pour bien comprendre SOAP, avec des exemples concrets pour chaque partie.

## 1. Structure de base d'un message SOAP

Un message SOAP est composé de plusieurs parties, généralement organisées en quatre sections principales : `Envelope`, `Header`, `Body`, et (facultatif) `Fault`.

```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Header>
        <!-- En-têtes, informations de sécurité, etc. -->
    </soap:Header>
    <soap:Body>
        <!-- Données principales de la requête ou réponse -->
    </soap:Body>
</soap:Envelope>
```

### Description des éléments

- **Envelope** : Encapsule tout le message SOAP. Il définit l'espace de noms pour que le message soit compris comme un message SOAP.
- **Header** : Section optionnelle où l'on peut ajouter des métadonnées ou des informations de routage, comme des informations de sécurité ou des instructions spécifiques.
- **Body** : Contient les données principales, c'est ici que se trouvent les informations de la requête ou de la réponse.
- **Fault** : Partie facultative qui s’ajoute dans le `Body` pour signaler les erreurs éventuelles.

## 2. Exemple simple d’une requête SOAP

Imaginons un service web SOAP qui fournit des informations météo. Pour demander la température d’une ville spécifique, nous envoyons une requête sous la forme d'un message SOAP.

### Requête SOAP

```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Body>
        <getTemperature xmlns="http://example.com/weather">
            <city>Paris</city>
        </getTemperature>
    </soap:Body>
</soap:Envelope>
```

### Explications

- L'élément `getTemperature` est la méthode appelée sur le service SOAP, définie par l’espace de noms `http://example.com/weather`.
- Dans le `Body`, nous incluons un paramètre `city` avec la valeur `Paris`, ce qui signifie que nous demandons la température pour Paris.

## 3. Exemple de réponse SOAP

Le service renvoie une réponse avec la température demandée, en suivant la même structure de base que la requête.

```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Body>
        <getTemperatureResponse xmlns="http://example.com/weather">
            <temperature>18</temperature>
            <unit>Celsius</unit>
        </getTemperatureResponse>
    </soap:Body>
</soap:Envelope>
```

### Explications

- `getTemperatureResponse` est la réponse de la méthode `getTemperature`.
- Le `Body` contient les éléments `temperature` et `unit`, avec la température de 18 degrés Celsius pour Paris.

## 4. Gestion des erreurs avec `Fault`

Quand une requête échoue, SOAP utilise l’élément `Fault` dans le `Body` pour décrire l’erreur. Par exemple, si le service ne trouve pas la ville demandée, il peut retourner une réponse comme celle-ci :

```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
    <soap:Body>
        <soap:Fault>
            <faultcode>soap:Client</faultcode>
            <faultstring>City not found</faultstring>
            <detail>
                <city>UnknownCity</city>
            </detail>
        </soap:Fault>
    </soap:Body>
</soap:Envelope>
```

### Explications

- `faultcode` : Code d'erreur standard. `soap:Client` indique que l'erreur provient de la demande du client.
- `faultstring` : Message descriptif de l’erreur, ici "City not found" (Ville introuvable).
- `detail` : Section optionnelle qui fournit des détails supplémentaires sur l’erreur, ici le nom de la ville inconnue.

## 5. Mise en œuvre en Java

En Java, SOAP peut être utilisé via l'API JAX-WS (Java API for XML Web Services). Imaginons un service Java pour obtenir la température. Voici comment pourrait être écrite la classe de service :

### Classe de Service SOAP

```java
import javax.jws.WebService;
import javax.jws.WebMethod;

@WebService
public class WeatherService {
    
    @WebMethod
    public String getTemperature(String city) {
        if ("Paris".equalsIgnoreCase(city)) {
            return "18 Celsius";
        } else {
            throw new RuntimeException("City not found");
        }
    }
}
```

### Explications

- L'annotation `@WebService` définit une classe comme un service web SOAP.
- L'annotation `@WebMethod` marque la méthode `getTemperature` comme une méthode accessible via SOAP.
- La méthode renvoie la température de Paris, et lance une exception si la ville est inconnue (ce qui se traduira par un `Fault` côté client).

### Appel du Service SOAP depuis un Client

Pour appeler un service SOAP depuis un client Java, on peut générer un proxy avec JAX-WS. Exemple :

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

## 6. Avantages et inconvénients de SOAP

**Avantages :**

- **Standardisé** : SOAP est un standard reconnu, compatible avec de nombreux systèmes.
- **Sécurité** : SOAP supporte des standards de sécurité avancés, comme WS-Security, pour l'authentification et l’intégrité des messages.
- **Interopérabilité** : SOAP est compatible avec de nombreuses plateformes et langages.

**Inconvénients :**

- **Complexité** : Les messages SOAP sont en XML et contiennent beaucoup d’informations supplémentaires, ce qui peut augmenter la charge réseau.
- **Rigidité** : SOAP suit des standards très stricts, ce qui peut rendre sa mise en œuvre plus rigide comparée à d'autres alternatives comme REST.

## Conclusion

SOAP est un protocole puissant mais souvent considéré comme complexe pour des applications modernes. Il est utile pour des environnements nécessitant une interopérabilité robuste, une sécurité renforcée et une gestion des erreurs structurée. Si vous travaillez dans un contexte d'entreprise, SOAP peut être un bon choix, mais pour des applications plus légères, des solutions comme REST (qui utilise JSON au lieu de XML) peuvent être plus adaptées.
