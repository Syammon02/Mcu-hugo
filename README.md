# Le MCU dans l'ordre

Checklist des 38 films du MCU dans l'ordre de sortie, avec suivi de progression synchronisé.

Site 100 % statique : un seul fichier, `index.html`. Aucun build.

## Suivi des films vus

- La progression est synchronisée dans Supabase (table `mcu_films_vus` du projet `ecepookbxkqgclxkhkcf`), par appareil : un identifiant anonyme est généré et gardé dans `localStorage` (`mcu-device-id`).
- `localStorage` sert de cache local et de repli hors ligne ; à la reconnexion, les coches locales non poussées sont renvoyées au serveur.

Schéma de la table (RLS activé, accès anonyme en select/insert/delete) :

```sql
create table public.mcu_films_vus (
  device_id text not null check (char_length(device_id) between 8 and 64),
  film_id   text not null check (film_id ~ '^f[0-9]{2}$'),
  vu_le     timestamptz not null default now(),
  primary key (device_id, film_id)
);
```

## Affiches des films

Chargées côté client, dans cet ordre :

1. **OMDb** (<https://www.omdbapi.com>) si une clé API est renseignée dans la constante
   `OMDB_KEY` en tête du script de `index.html` (clé gratuite sur le site OMDb) ;
2. **Wikipédia** (API `pageimages`, une requête pour tous les films) pour les films
   qu'OMDb n'a pas, ou pour tout si `OMDB_KEY` est vide ;
3. sans réseau ni résultat, les tuiles colorées d'origine restent affichées.

Le résultat est mis en cache 30 jours dans `localStorage` (`mcu-affiches-v2`).

## Déploiement sur Vercel

1. Sur <https://vercel.com/new>, importer le dépôt GitHub `Syammon02/Mcu-hugo`.
2. Framework preset : **Other**. Aucune commande de build, aucun dossier de sortie à
   configurer (site statique servi tel quel).
3. Chaque push sur la branche de production déclenche ensuite un déploiement automatique.
