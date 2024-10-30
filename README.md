
# README de Configuration Nginx

Ce document fournit une explication de la configuration Nginx utilisée pour mettre en place un proxy inverse avec un backend en amont pour l'équilibrage de charge entre deux serveurs.

## Décomposition de la Configuration Nginx

### Bloc Upstream

```nginx
upstream backend {
    server localhost:5000 weight=2; # Serveur principal avec une priorité plus élevée
    server localhost:5001 backup;   # Serveur secondaire en tant que sauvegarde
}
```

- **`upstream backend { ... }`** : Ce bloc définit un groupe de serveurs backend que Nginx peut utiliser pour l'équilibrage de charge. Le nom `backend` peut être référencé plus tard dans la configuration.

- **`server localhost:5000 weight=2;`** : 
  - **`server`** : Cette directive spécifie un serveur auquel Nginx peut transférer des requêtes.
  - **`localhost:5000`** : Cela indique que le premier serveur s'exécute sur la machine locale, à l'écoute du port 5000.
  - **`weight=2`** : Ce paramètre attribue une priorité plus élevée à ce serveur. Lorsque Nginx distribue les requêtes, ce serveur recevra deux fois plus de requêtes que le poids par défaut (qui est 1).

- **`server localhost:5001 backup;`** : 
  - **`localhost:5001`** : Cela spécifie le deuxième serveur, qui fonctionne également sur la machine locale mais à l'écoute du port 5001.
  - **`backup`** : Cette directive marque ce serveur comme une sauvegarde. Il ne sera utilisé que si le serveur principal (localhost:5000) est indisponible.

### Bloc Serveur

```nginx
server {
    listen 80;
    server_name main_server.local backup_server.local;
}
```

- **`server { ... }`** : Ce bloc définit un contexte de serveur pour gérer les requêtes entrantes.

- **`listen 80;`** : Cette directive indique à Nginx d'écouter les connexions entrantes sur le port 80, qui est le port par défaut pour le trafic HTTP.

- **`server_name main_server.local backup_server.local;`** : 
  - **`server_name`** : Cette directive spécifie les noms de domaine qui seront gérés par ce bloc de serveur.
  - **`main_server.local`** : Le nom de domaine principal pour ce serveur.
  - **`backup_server.local`** : Un nom de domaine supplémentaire qui sera servi par le même bloc.

### Bloc de Localisation

```nginx
    location / {
        proxy_pass http://main_server.local;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

- **`location / { ... }`** : Ce bloc définit comment Nginx doit répondre aux requêtes pour l'URL racine (`/`). Les règles définies ici s'appliqueront à toutes les requêtes entrantes.

- **`proxy_pass http://main_server.local;`** : Cette directive indique à Nginx de transférer la requête au serveur en amont spécifié (`main_server.local`). Cela agit effectivement comme un proxy inverse.

- **`proxy_set_header Host $host;`** : Cela définit l'en-tête `Host` dans la requête proxy avec la valeur de l'en-tête `Host` de la requête d'origine. Cela garantit que le serveur backend connaît le nom d'hôte d'origine demandé.

- **`proxy_set_header X-Real-IP $remote_addr;`** : Cela définit l'en-tête `X-Real-IP` à l'adresse IP réelle du client (`$remote_addr`). Cela est utile pour la journalisation et pour que le serveur backend connaisse l'adresse IP réelle du client.

- **`proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;`** : Cela ajoute l'adresse IP du client à l'en-tête `X-Forwarded-For`. S'il y a plusieurs proxies, cet en-tête peut contenir une chaîne d'adresses IP.

- **`proxy_set_header X-Forwarded-Proto $scheme;`** : Cela définit l'en-tête `X-Forwarded-Proto` au protocole utilisé dans la requête d'origine (HTTP ou HTTPS). Cela aide le backend à comprendre comment la requête a été effectuée.

---

Cette configuration met efficacement en place un simple équilibrage de charge avec un serveur backend principal et un serveur de sauvegarde, garantissant que le trafic est acheminé de manière appropriée tout en préservant des informations importantes sur le client. Si vous avez des questions ou avez besoin de modifications supplémentaires, n'hésitez pas à demander !