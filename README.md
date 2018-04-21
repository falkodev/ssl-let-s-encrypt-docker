# SSL : Let's encrypt + Docker

Il y a quelques années, la génération d'un certificat SSL \(TLS pour les puristes, mais tout le monde dit SSL, donc on va garder ça pour cet article\) était complexe et surtout payant. Ca peut toujours être le cas d'ailleurs.

Mais en 2015, un consortium d'acteurs importants du net, tels que la fondation Mozilla, Cisco, OVH et d'autres ont fondé _Let's encrypt_, une autorité de certification délivrant gratuitement et facilement des certificats pour aider à une meilleure sécurisation des sites internet.

Il devenait donc plus simple d'avoir son site en HTTPS. Mais avec un site tournant sur Docker en production, comment demander la création et le renouvellement du fameux certificat SSL/TLS ?

## Génération du certificat

Différentes méthodes existent sur internet, et le[ site officiel de _Let's encrypt_](https://certbot.eff.org/) vise à aider les utilisateurs en proposant un bot pour cela.

![](/assets/certbot.png)Mais rien pour Docker. On pourrait toujours générer un certificat manuellement, et l'importer dans le conteneur Docker. Mais une belle automatisation des familles ce serait mieux.

Heureusement, [Quay.io](https://quay.io/), une entreprise spécialisée dans la gestion de conteneurs Docker a construit une image prête à l'emploi et la laisse à disposition du grand public. Pour générer un certificat _Let's encrypt_, il faut répondre à plusieurs questions comme le ou les noms de domaine à gérer, l'adresse email à contacter, accepter les conditions générales, ...

L'image Docker de Quay permet de tout passer en paramètres d'une commande pour générer le certificat très simplement.

```js
docker run -it --rm -p 443:443 -p 80:80 --name letsencrypt  \
-v "/path/to/a/letsencrypt/folder:/etc/letsencrypt" \
-v "/path/to/a/temp/folder:/var/lib/letsencrypt" \ 
quay.io/letsencrypt/letsencrypt:latest auth --standalone --agree-tos --renew-by-default \ 
-d tarlao.fr -d www.tarlao.fr -m email_adress@to_contact
```

L'image Docker `quay.io/letsencrypt/letsencrypt` se télécharge, le certificat se génère et est stocké dans `/etc/letsencrypt` de la machine hôte. Ensuite, dans le serveur web, on fait référence au certificat pour autoriser HTTPS. Par exemple, avec un serveur Nginx:

```js
ssl_certificate /etc/letsencrypt/live/tarlao.fr/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/tarlao.fr/privkey.pem;
```

On peut vérifier que le certificat est bien validé par _Let's encrypt_ en allant sur [https://www.ssllabs.com/ssltest/](https://www.ssllabs.com/ssltest/)

![](/assets/ssllabs.png)

## Renouvellement du certificat

Le certificat est valable 3 mois, et au bout de 2,_ Let's encrypt_ avertit sur l'adresse email de contact qu'il faut penser à le renouveler. On peut le faire manuellement, mais on peut aussi se faire un simple script bash qui va lancer la commande décrite ci-dessus et programmer un cron mensuel sur son serveur. Et ainsi, plus besoin de se préoccuper d'une expiration oubliée.

