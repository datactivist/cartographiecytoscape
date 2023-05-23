## besoins

besoin_short,besoin,besoin_0506,nb_usages,usages,polpub,acteurs,dgaln_interlocuteurs,bdd,dgfip_bureaux,dgfip_interlocuteurs,dgfip_actions,dgfip_commentaires,dgaln_avancement,dgaln_commentaires,dgaln_priorite

CREATE TABLE besoins_raw (besoin_short varchar,besoin text,besoin_0506 varchar,nb_usages int,usages text,polpub text,acteurs text,dgaln_interlocuteurs text,bdd text,dgfip_bureaux text,dgfip_interlocuteurs text,dgfip_actions text,dgfip_commentaires text,dgaln_avancement text,dgaln_commentaires text,dgaln_priorite text);

CREATE TABLE besoins (besoin_short varchar,besoin text,besoin_0506 varchar,nb_usages int,usages varchar[],polpub text[],dgaln_acteurs varchar[],dgaln_interlocuteurs text[],bdd varchar[],dgfip_bureaux varchar[],dgfip_interlocuteurs text[],dgfip_actions text,dgfip_commentaires text,dgaln_avancement text,dgaln_commentaires text,dgaln_priorite text);

INSERT INTO besoins SELECT besoin_short,besoin,besoin_0506,nb_usages,string_to_array(usages,',')::varchar[],string_to_array(polpub,','),string_to_array(acteurs,',')::varchar[],string_to_array(dgaln_interlocuteurs,','),string_to_array(bdd,',')::varchar[],string_to_array(dgfip_bureaux,',')::varchar[],string_to_array(dgfip_interlocuteurs,',')::varchar[],dgfip_actions,dgfip_commentaires,dgaln_avancement,dgaln_commentaires,dgaln_priorite FROM besoins_raw;

## Usages

usage_short,usage,polpub,commentaires

CREATE TABLE usages_raw (usage_short varchar,usage text,polpub text,commentaires text);

CREATE TABLE usages (usage_short varchar,usage text,polpub text[],commentaires text);

INSERT INTO usages SELECT usage_short,usage,string_to_array(polpub,','),commentaires FROM usages_raw;

## BDD

bdd,dgfip_bureaux,description,diffusion,commentaires

CREATE TABLE bdd (bdd varchar,dgfip_bureaux text,description text,diffusion varchar,commentaires text)

## Acteurs

bureau,description,direction,contacts,mail

CREATE TABLE acteurs_raw (bureau varchar,description text,direction varchar, ministere varchar,contacts text,mail text);

CREATE TABLE acteurs (bureau varchar,description text,direction varchar,ministere varchar,contacts text[],mail text);

INSERT INTO acteurs SELECT bureau,description,direction,ministere,string_to_array(contacts,','),mail FROM acteurs_raw;

## Links

with besoins_bdd as (select besoin_short, trim(unnest(bdd)) as bdd from besoins),
besoins_dgaln_acteurs as (select besoin_short, trim(unnest(dgaln_acteurs)) as dgaln_acteurs from besoins),
besoins_dgfip_bureaux as (select besoin_short, trim(unnest(dgfip_bureaux)) as dgfip_bureaux from besoins),
besoins_usages as (select besoin_short, trim(unnest(usages)) as usages from besoins)
SELECT b.besoin_short as in, bd.bdd as out, 'besoin_bdd' as link FROM besoins_bdd b JOIN bdd bd ON bd.bdd=b.bdd
UNION
SELECT b.besoin_short as in, a.bureau as out, 'besoin_dgaln' as link FROM besoins_dgaln_acteurs b JOIN acteurs a ON a.bureau=b.dgaln_acteurs
UNION
SELECT b.besoin_short as in, a.bureau as out, 'besoin_dgfip' as link FROM besoins_dgfip_bureaux b JOIN acteurs a ON a.bureau=b.dgfip_bureaux
UNION
SELECT b.besoin_short as in, u.usage_short as out, 'besoin_usage' as link FROM besoins_usages b JOIN usages u ON u.usage=b.usages
UNION
SELECT u.usage_short as in, trim(unnest(u.polpub)) as out, 'usage_polpub' as link from usages u
UNION
SELECT bd.bdd as in, bd.dgfip_bureaux as out, 'bdd_producteur' as link from bdd bd where bd.dgfip_bureaux is not null;

## Nodes

select b.besoin_short as node, 'besoin' as nature, besoin_0506 as echeance, b.besoin as description from besoins b
union
select a.bureau as node, 'acteur_mte' as nature, null as echeance, a.description from acteurs a where a.ministere='MTE'
union
select a.bureau as node, 'acteur_mefsin' as nature, null as echeance, a.description from acteurs a where a.ministere='MEFSIN'
union
select a.bureau as node, 'acteur_tiers' as nature, null as echeance, a.description from acteurs a where a.ministere is null
union
select bd.bdd as node, 'bdd' as nature, null as echeance, bd.description from bdd bd
union
select u.usage_short as node, 'usage' as nature, null as echeance, u.usage as description from usages u
union
select distinct trim(unnest(u.polpub)) as node, 'polpub' as nature, null as echeance, null as description from usages u;
