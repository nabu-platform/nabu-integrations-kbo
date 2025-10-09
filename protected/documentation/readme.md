# KBO

## Concepts

- enterprise: an actual organisation
- establishment: a single physical location for an organisation, e.g. a warehouse, office,...
- branch: an establishment that has a legal representative and can thus bind the enterprise to commitments
- entity: any of the above

# Webservices

Test endpoint: https://kbopub-acc.economie.fgov.be/kbopubws110000/services/wsKBOPub
Prd endpoint: https://kbopub.economie.fgov.be/kbopubws110000/services/wsKBOPub

# Files

KBO has [multiple files](https://kbopub.economie.fgov.be/kbo-open-data/login) in its full download, some contain masterdata, others contain operational data.

By far the biggest dataset are the NACE codes for each company. If you do not need those, you can reduce your processing and storage needs significantly by not processing them.

Note that this is written before october 2025 at which point they will switch to daily updates. It is unclear how things will change at that point. At the very least they will need to revisit their file name convention for the downloads.

## Security

You need to [request an account](https://kbopub.economie.fgov.be/kbo-open-data/signup?form) but other than that the access is free.

Unfortunately it seems to have been designed for "manual" downloads rather than automatic.
For login you need to do a call to https://kbopub.economie.fgov.be/kbo-open-data/static/j_spring_security_check using form-encoded parameters

```
POST /kbo-open-data/static/j_spring_security_check
Host: kbopub.economie.fgov.be
Content-Type: application/x-www-form-urlencoded

j_username=<username>&j_password=<password>
```

As a response you will get a bunch of cookies back, it seems that you need these two to actually do the download:

- JSESSIONID
- KSESSIONID-KBOPUB

Then proceed to download the data (providing the cookies):

- full: https://kbopub.economie.fgov.be/kbo-open-data/affiliation/xml/files/KboOpenData_0137_2025_07_Full.zip
- update: https://kbopub.economie.fgov.be/kbo-open-data/affiliation/xml/files/KboOpenData_0137_2025_07_Update.zip

The 0137 in the file name is incremental, the month is self explanatory.

## Full download

Note that file sizes that are mentioned are based on a snapshot taken in june 2025.

### meta.csv

Size: 149bytes
Contents: basic metadata about the download itself, this can usually be ignored:

```
"Variable","Value"
"SnapshotDate","30-05-2025"
"ExtractTimestamp","01-06-2025 07:07:49"
"ExtractType","full"
"ExtractNumber","136"
"Version","1.0.0"
```

### branch.csv

Size: 307.3kb
Contents: mapping from establishments of an enterprise to the enterprise itself (including the start date)

```
"Id","StartDate","EnterpriseNumber"
"9.000.006.626",01-09-1995,"0257.883.408"
"9.000.008.210",01-01-1968,"0400.051.358"
"9.000.008.705",01-01-1968,"0400.067.095"
"9.000.009.594",01-01-1968,"0400.150.140"
```

### code.csv

Size: 1.9mb
Contents: masterdata categories in multiple languages:

```
"Category","Code","Language","Description"
"ActivityGroup","001","FR","Activités TVA"
"ActivityGroup","001","NL","BTW-activiteiten"
"ActivityGroup","002","FR","Activités EDRL"
"ActivityGroup","002","NL","EDRL-activiteiten"
...
"Classification","ANCI","FR","Activité auxiliaire"
"Classification","ANCI","NL","Hulpactiviteit"
"Classification","MAIN","FR","Activité principale"
"Classification","MAIN","NL","Hoofdactiviteit"
"Classification","SECO","FR","Activité secondaire"
"Classification","SECO","NL","Nevenactiviteit"

"ContactType","EMAIL","FR","Adresse e-mail"
"ContactType","EMAIL","NL","E-mailadres"
"ContactType","TEL","FR","Numéro de téléphone"
"ContactType","TEL","NL","Telefoonnummer"
"ContactType","WEB","FR","Adresse web"
"ContactType","WEB","NL","Webadres"

"EntityContact","BRA","FR","Succursale"
"EntityContact","BRA","NL","Bijkantoor"
"EntityContact","ENT","FR","Entreprise"
"EntityContact","ENT","NL","Onderneming"
"EntityContact","EST","FR","Unité d'établissement"
```

The categories (as of 2025) are:

```
ActivityGroup
Category
Classification
ContactType
EntityContact
JuridicalForm
JuridicalSituation
Language
Nace2003
Nace2008
Nace2025
Status
TypeOfAddress
TypeOfDenomination
TypeOfEnterprise
```

Script to list the categories:

```glue
string code = file.read("code.csv")
lines = split("\n", file.read("code.csv"))
categories = replace("^\"([^\"]+).*", "$1", lines)
echo(unwrap(sort(series: unique(categories))))
```

Note that the nace codes take up the absolute bulk of the entries. Out of the 21500 entries, 20876 of them are nace codes which leaves only 624 other entries.

### contact.csv

Size: 33.3mb
Contents: all the contact information for the entities:

```
"EntityNumber","EntityContact","ContactType","Value"
"0200.362.210","ENT","EMAIL","officiel.ic-inbw@inbw.be"
"0200.362.408","ENT","TEL","02 315 13 51"
"0200.362.408","ENT","EMAIL","info@isbw.be"
"0201.105.843","ENT","EMAIL","officiel.ic-idea@idea.be"
"0201.107.922","ENT","EMAIL","officiel.ic-irsia@irsia.be"
"0201.310.929","ENT","TEL","089/32.39.50"
"0201.310.929","ENT","FAX","089/36.17.97"
"0201.310.929","ENT","WEB","extranet.iglimburg.be/"
"0201.400.011","ENT","EMAIL","officiel.ic-bepexpansioneconomique@bep.be"
"0201.400.110","ENT","TEL","083611205"
```

Relevant masterdata:

```
"EntityContact","BRA","FR","Succursale"
"EntityContact","BRA","NL","Bijkantoor"
"EntityContact","ENT","FR","Entreprise"
"EntityContact","ENT","NL","Onderneming"
"EntityContact","EST","FR","Unité d'établissement"
"EntityContact","EST","NL","Vestigingseenheid"

"ContactType","EMAIL","FR","Adresse e-mail"
"ContactType","EMAIL","NL","E-mailadres"
"ContactType","TEL","FR","Numéro de téléphone"
"ContactType","TEL","NL","Telefoonnummer"
"ContactType","WEB","FR","Adresse web"
"ContactType","WEB","NL","Webadres"
```

### establishment.csv

Size: 69.6mb 
Contents: all establishments linked to a particular enterprise

```
"EstablishmentNumber","StartDate","EnterpriseNumber"
"2.000.000.339",01-11-1974,"0403.449.823"
"2.000.000.438",01-09-1964,"0403.590.274"
"2.000.001.032",17-01-1967,"0403.174.758"
"2.000.002.022",01-04-1963,"0401.729.854"
```

### enterprise.csv

Size: 89.0mb
Contents: all the enterprises with a minimum set of metadata

```
"EnterpriseNumber","Status","JuridicalSituation","TypeOfEnterprise","JuridicalForm","JuridicalFormCAC","StartDate"
"0200.065.765","AC","000","2","416",,09-08-1960
"0200.068.636","AC","000","2","417",,16-02-1923
"0200.171.970","AC","000","2","116",,01-01-1968
"0200.245.711","AC","012","2","116",,01-01-1922
"0200.305.493","AC","000","2","416",,19-03-1962
"0200.362.210","AC","000","2","706",,26-02-1968
"0200.362.408","AC","000","2","706",,01-01-1968
"0200.420.410","AC","000","2","051",,01-01-1968
"0200.420.608","AC","000","2","124",,01-01-1968
"0200.448.421","AC","000","2","124",,02-08-1963
```

Relevant masterdata:

```
"Status","AC","FR","Actif"
"Status","AC","NL","Actief"

"TypeOfEnterprise","0","FR","Inconnu"
"TypeOfEnterprise","0","NL","Onbekend"
"TypeOfEnterprise","1","FR","Personne physique"
"TypeOfEnterprise","1","NL","Natuurlijk Persoon"
"TypeOfEnterprise","2","FR","Personne morale"
"TypeOfEnterprise","2","NL","Rechtspersoon"

"JuridicalSituation","000","DE","gewöhnlicher Zustand"
"JuridicalSituation","000","FR","Situation normale"
"JuridicalSituation","000","NL","Normale toestand"
"JuridicalSituation","001","DE","juristische Gründung"
"JuridicalSituation","001","FR","Création juridique"
"JuridicalSituation","001","NL","Juridische oprichting"
"JuridicalSituation","002","DE","Verlängerung"
"JuridicalSituation","002","FR","Prorogation"
"JuridicalSituation","002","NL","Verlenging"
(there are too many entries to list them all here)

"JuridicalForm","001","DE","Europäische Genossenschaft"
"JuridicalForm","001","FR","Société coopérative européenne"
"JuridicalForm","001","NL","Europese Coöperatieve Vennootschap"
"JuridicalForm","002","DE","Organismus für die Finanzierung von Pensionen"
"JuridicalForm","002","FR","Organisme de financement de pensions"
"JuridicalForm","002","NL","Organisme voor de Financiering van Pensioenen"
"JuridicalForm","003","DE","Mehrwertsteuereinheit"
...
"JuridicalForm","124","DE","Öffentliche Einrichtung"
"JuridicalForm","124","FR","Etablissement public"
"JuridicalForm","124","NL","Openbare instelling"
(there are too many entries to list them all here)
```

There does not appear to be an "inactive" status.

### denomination.csv

Size: 152mb
Contents: the various names of the entities

```
"EntityNumber","Language","TypeOfDenomination","Denomination"
"0200.065.765","2","001","Intergemeentelijke Vereniging Veneco"
"0200.065.765","2","002","Veneco"
"0200.068.636","2","001","Farys"
"0200.171.970","0","001","Sanatorium-Hospitaal van Lemberge"
"0200.245.711","2","001","Intercommunaal Sanatorium Denderoord"
"0200.245.711","2","002","DENDEROORD"
"0200.305.493","2","001","Intergemeentelijk Samenwerkingsverband voor ruimtelijke ordening en socio-economische expansie"
"0200.305.493","2","002","SOLVA"
"0200.362.210","1","001","in BW Association Intercommunale"
"0200.362.210","1","002","in B.W."
"0200.362.408","1","001","Intercommunale Sociale du Brabant wallon"
"0200.362.408","1","002","I.S.B.W."
"0200.420.410","0","001","Congregatie der Gasthuiszusters-Augustinessen van Diest"
```

The relevant masterdata:

```
"TypeOfDenomination","001","FR","Dénomination"
"TypeOfDenomination","001","NL","Naam"
"TypeOfDenomination","002","FR","Abréviation"
"TypeOfDenomination","002","NL","Afkorting"
"TypeOfDenomination","003","FR","Dénomination commerciale"
"TypeOfDenomination","003","NL","Commerciële naam"
"TypeOfDenomination","004","FR"," Dénomination de la succursale"
"TypeOfDenomination","004","NL","Naam van het bijkantoor"

"Language","0","FR","inconnu"
"Language","0","NL","Onbekend"
"Language","1","FR","français"
"Language","1","NL","Frans"
"Language","2","FR","néerlandais"
"Language","2","NL","Nederlands"
"Language","3","FR","allemand"
"Language","3","NL","Duits"
"Language","4","FR","anglais"
"Language","4","NL","Engels"
```

### address.csv

Size: 297.4mb
Contents: the addresses for the entities

```
"EntityNumber","TypeOfAddress","CountryNL","CountryFR","Zipcode","MunicipalityNL","MunicipalityFR","StreetNL","StreetFR","HouseNumber","Box","ExtraAddressInfo","DateStrikingOff"
"0200.065.765","REGO",,,"9070","Destelbergen","Destelbergen","Panhuisstraat","Panhuisstraat","1","","",
"0200.068.636","REGO",,,"9000","Gent","Gent","Stropstraat","Stropstraat","1","","",
"0200.171.970","REGO",,,"9000","Gent","Gent","Brabantdam","Brabantdam","101","","",
"0200.245.711","REGO",,,"9500","Geraardsbergen","Geraardsbergen","Hoge Buizemont","Hoge Buizemont","247","","",
"0200.305.493","REGO",,,"9520","Sint-Lievens-Houtem","Sint-Lievens-Houtem","Gentsesteenweg","Gentsesteenweg","1B","","",
"0200.362.210","REGO",,,"1400","Nivelles","Nivelles","Rue de la Religion","Rue de la Religion","10","","",
"0200.362.408","REGO",,,"1332","Rixensart","Rixensart","Rue du Cerf","Rue du Cerf","200","","",
"0200.420.410","REGO",,,"3290","Diest","Diest","Michel Theysstraat","Michel Theysstraat","18","","",
"0200.420.608","REGO",,,"8970","Poperinge","Poperinge","Ieperstraat","Ieperstraat","134","","",
"0200.448.421","REGO",,,"8500","Kortrijk","Kortrijk","Budastraat(Kor)","Budastraat(Kor)","37","","",
"0200.450.005","REGO",,,"8000","Brugge","Brugge","Woensdagmarkt","Woensdagmarkt","6","","",
"0200.762.878","REGO",,,"2850","Boom","Boom","Colonel Silvertopstraat","Colonel Silvertopstraat","15","","",
"0200.881.951","REGO",,,"1731","Asse","Asse","Brusselsesteenweg","Brusselsesteenweg","617","","",
"0200.882.050","REGO",,,"1000","Brussel","Bruxelles","Eikstraat","Rue du Chêne","16","","",
"0201.105.843","REGO",,,"7000","Mons","Mons","Rue de Nimy","Rue de Nimy","53","","",
...
"0400.051.358","REGO","Duitsland (Bondsrep.)","Allemagne (Rép.féd.)","","West-Duitsland","West-Duitsland","Stolberg  519 (Rheinland)","Stolberg  519 (Rheinland)","","","",
...
"0400.067.095","REGO","Frankrijk","France","62000","Arras","Arras","Rue des Jongleurs","Rue des Jongleurs","2","","",
...
"0400.357.897","REGO",,,"1420","Braine-l'Alleud","Braine-l'Alleud","Chaussée d'Alsemberg","Chaussée d'Alsemberg","421","","",13-11-2023
```

Note that country does not appear to filled in for belgian addresses while zipcode etc are not guaranteed in other countries.

Relevant masterdata:

```
"TypeOfAddress","ABBR","FR","Succursale"
"TypeOfAddress","ABBR","NL","Bijkantoor"
"TypeOfAddress","BAET","FR","Unité d'établissement"
"TypeOfAddress","BAET","NL","Vestigingseenheid"
"TypeOfAddress","OBAD","FR","Première unité d'établissement active"
"TypeOfAddress","OBAD","NL","Oudste actieve vestigingseenheid"
"TypeOfAddress","REGO","FR","Siège"
"TypeOfAddress","REGO","NL","Zetel"
```

It seems that the "dateStrikingOff" means the address is no longer valid since that date. It is merely maintained for historical correctness.
Because we handle delete delta's by "soft deleting", this creates a natural deletion timestamp which is why we don't have an additional field.

Note however that in the initial load we can choose to sync the already inactive as well or only start with active ones.
When "updating" they will delete the address and reinsert it with a dateStrikingOff. This date _is_ a more precise date for deletion than what we have (which is either the time of running or the date in the meta file) so we should still get that to register a more precise delete statement.

### activity.csv

Size: 1.6gb
Contents: the nace code listings for entities

```
"EntityNumber","ActivityGroup","NaceVersion","NaceCode","Classification"
"0200.065.765","006","2025","84130","MAIN"
"0200.065.765","001","2003","70111","MAIN"
"0200.065.765","001","2025","68121","MAIN"
"0200.065.765","006","2008","84130","MAIN"
"0200.065.765","001","2008","41101","MAIN"
"0200.068.636","006","2025","36000","MAIN"
"0200.068.636","001","2003","41000","MAIN"
"0200.068.636","001","2025","93110","SECO"
"0200.068.636","001","2025","36000","MAIN"
"0200.068.636","001","2025","93299","SECO"
"0200.068.636","001","2025","93126","SECO"
"0200.068.636","006","2008","36000","MAIN"
"0200.068.636","001","2008","93110","SECO"
"0200.068.636","001","2008","93299","SECO"
"0200.068.636","001","2008","36000","MAIN"
"0200.068.636","001","2008","93126","SECO"
"0200.305.493","006","2025","84130","MAIN"
"0200.305.493","001","2003","45213","SECO"
"0200.305.493","001","2025","41003","SECO"
```

Relevant masterdata (apart from nace codes itself):

```
"ActivityGroup","001","FR","Activités TVA"
"ActivityGroup","001","NL","BTW-activiteiten"
"ActivityGroup","002","FR","Activités EDRL"
"ActivityGroup","002","NL","EDRL-activiteiten"
"ActivityGroup","003","FR","Activités"
"ActivityGroup","003","NL","Activiteiten"
"ActivityGroup","004","FR","Activités de la fonction publique fédérale"
"ActivityGroup","004","NL","Activiteiten federaal openbaar ambt"
"ActivityGroup","005","FR","Activités ONSSAPL"
"ActivityGroup","005","NL","RSZPPO-activiteiten"
"ActivityGroup","006","FR","Activités ONSS"
"ActivityGroup","006","NL","RSZ-activiteiten"
"ActivityGroup","007","FR","Activités de l'enseignement subventionné"
"ActivityGroup","007","NL","Activiteiten gesubsideerd onderwijs"

"Classification","ANCI","FR","Activité auxiliaire"
"Classification","ANCI","NL","Hulpactiviteit"
"Classification","MAIN","FR","Activité principale"
"Classification","MAIN","NL","Hoofdactiviteit"
"Classification","SECO","FR","Activité secondaire"
"Classification","SECO","NL","Nevenactiviteit"
```

The nace code works slightly differently, in the code tabel the code and version are combined:

```
"Nace2003","92611","FR","Gestion et exploitation de centres sportifs"
"Nace2003","92611","NL","Beheer en exploitatie van sportcentra"
...
"Nace2008","06200","FR","Extraction de gaz naturel"
"Nace2008","06200","NL","Winning van aardgas"
...
"Nace2025","84220","FR","Défense"
"Nace2025","84220","NL","Defensie"
```

## Update download

Once a month (from october onwards probably once per day), an update zip is provided.
The file sizes are based on the update of july 2025.

Where relevant updates are expressed as insert + delete, for instance in the full you might see:

```
address.csv:"0220.815.847","REGO",,,"4790","Burg-Reuland","Burg-Reuland","Bergstraße, Dürler","Bergstraße, Dürler","6","","",
denomination.csv:"0220.815.847","3","001","Kirchenfabrik Sankt-Matthias in Dürler (WL - Burg-Reuland)"
enterprise.csv:"0220.815.847","AC","000","2","124",,01-01-1980
```

But because the address was updated, the delta for that same month contains this:

```
address_delete.csv:"0220.815.847"
address_insert.csv:"0220.815.847","REGO",,,"4790","Burg-Reuland","Burg-Reuland","Wallhäuser Straße, Dürler","Wallhäuser Straße, Dürler","31","","",
```

This means it is _very_ important to first process the deletes and only _then_ the inserts.

### branch_delete.csv

Size: 293 bytes
Contents: the ids to delete

```
"Id"
"9.000.161.430"
"9.000.928.918"
"9.001.057.986"
"9.001.215.562"
"9.001.293.162"
```

### branch_insert.csv

Size: 456 bytes
Contents: same as branch.csv

### code.csv

Size: 1.9mb
Contents: exactly the same as the full upload

### contact_delete.csv

Size: 49.8kb
Contents: contacts to delete by entity number

```
"EntityNumber"
"0207.437.468"
"0211.122.478"
"0211.123.270"
"0211.139.306"
"0211.168.802"
"0211.177.017"
```

### contact_insert.Csv

Size: 346.9kb
Contents: same as contact.csv

### establishment_delete.csv

Size: 182.9kb
Contents: delete by establishment number

```
"EstablishmentNumber"
"2.000.164.645"
"2.000.195.032"
"2.000.211.462"
"2.000.267.880"
"2.000.306.482"
```

### establishment_insert.csv

Size: 593.7kb
Content: same as establishment.csv

### enterprise_delete.csv

Size: 208.5kb
Contents: delete by enterprise number

```
"EnterpriseNumber"
"0202.962.701"
"0211.122.478"
"0211.168.802"
"0211.169.097"
"0211.169.394"
"0211.176.720"
"0211.177.017"
```

### enterprise_insert.csv

Size: 788.5kb
Contents: same as enterprise.csv

### denomination_delete.csv

Size: 347.8kb
Contents: the denominations to delete by entity number

```
"EntityNumber"
"0211.122.478"
"0211.133.663"
"0211.168.802"
"0211.169.097"
"0211.169.394"
```

### denomination_insert.csv

Size: 1.4mb
Contents: same as denomination.csv

### address_delete.csv

Size: 457.7kb
Contents: the addresses to delete

```
"EntityNumber"
"0211.435.848"
"0220.724.686"
"0237.224.881"
"0400.140.539"
"0400.258.226"
```

### address_insert.csv

Size: 11.1mb
Contents: same as address.csv

### activity_delete.csv

Size: 665.9kb
Contents: activities to delete by entity number

```
"EntityNumber"
"0206.565.953"
"0207.401.935"
"0207.505.863"
```

### activity_insert.csv

Size: 17.8mb
Contents: same as the "activity.csv"

