#### Eleccions generals 
## APARTAT 1: Importació de les dades
### Modificació de l'script sql
	Per poder posar la taula de persones hem hagut de posar persona_id com a AUTO_INCREMENT i el dni com a NULLABLE i sense la UK
### Introducció de dades bàsiques
	Hem hagut d'insertar una elecció de la seguent manera:
	INSERT INTO eleccions(nom,data)
		VALUES ('Elecciones al Congreso de los Diputados','2016-06-26');

### A tots els programes es repeteix la part seguent que consisteix en connectar-se al servidor i per tant ho documentarem una vegada:
	import mysql.connector

	mydb = mysql.connector.connect(
	  host="192.168.56.103",
	  user="perepi",
	  passwd="pastanaga",
	  database="mydb"
	)
### Importació de comunitats autònomes, províncies i municipis:
	Comunitats autónomes:
	
	mycursor = mydb.cursor()
	for i in f:
	    if i[9:14] == "99999":
		continue
	    elif i[11:13] == "99":
		nom_CA = i[14:63]
		cod_CA = i[9:11]
		print(nom_CA.strip(),",", cod_CA)
		sql = "INSERT INTO comunitats_autonomes (nom, codi_ine) VALUES (%s, %s)"
		val = [nom_CA.strip(), cod_CA]
		mycursor.execute(sql, val)
	    else:
		continue
	mydb.commit()
	
	Províncies:
	mycursor = mydb.cursor()
	for i in f:
	    if i[9:14] == "99999":
		continue
	    elif i[11:13] != "99":
		nom_prov = i[14:63]
		cod_ine = i[11:13]
		cod_ca = i[9:11]
		escons = i[149:154]
		print(nom_prov)
		query = ("SELECT comunitat_aut_id FROM comunitats_autonomes WHERE %s = codi_ine")
		dades = [cod_ca]
		mycursor.execute(query, dades)
		com_aut_id = mycursor.fetchone()
		com_aut_id = " ".join(map(str,com_aut_id))
		print(nom_prov, cod_ine, cod_ca, escons, com_aut_id)
		sql = "INSERT INTO provincies (comunitat_aut_id, nom, codi_ine, num_escons) VALUES (%s, %s, %s, %s)"
		val = [com_aut_id, nom_prov.strip(), cod_ine, escons]
		mycursor.execute(sql, val)
	    else:
		continue
	mydb.commit()
	
	Municipis:
	
	Hi han una sèrie de valors del codi INE dels municipis repetits i amb el mateix codi de districte, 
	i per aixó hem optat per eliminar la constraint unique de la columna codi_ine amb la seguent sentència:
	Alter table municipis 
		DROP CONSTRAINT uk_municipis_codi_ine;
	
	mycursor = mydb.cursor()
	for i in f:
	    nom= i[18:118]
	    INE_mun= i[13:16]
	    INE_prov= i[11:13]
	    Distr= i[16:18]
	    query = ("SELECT provincia_id FROM provincies WHERE %s = codi_ine")
	    dades = [INE_prov]
	    mycursor.execute(query, dades)
	    prov_id = mycursor.fetchone()
	    prov_id = " ".join(map(str, prov_id))
	    print(nom.strip(), INE_prov, INE_mun, Distr, prov_id)
	    sql = "INSERT INTO municipis (nom, codi_ine, provincia_id, districte) VALUES (%s, %s, %s, %s)"
	    val = [nom.strip(), INE_mun, prov_id, Distr]
	    mycursor.execute(sql, val)
	mydb.commit()

### Importació de partits polítics/candidatures
Per omplir la taula de candidatures, hem hagut d'utilitzar les dades del fixer 03021606.DAT i executar el següent script:

	mycursor = mydb.cursor()

	archivo = open(r"C:\Users\david\Desktop\Pràctica BDD\candidaturas.dat")

	Candidatures:

	for x in archivo:
	    codi_candidatura = x[8:14]
	    sigles_candidatura = x[14:64]
	    denom_candidatura = x[64:214]
	    cod_acom_provi = x[214:220]
	    cod_acom_auto = x[220:226]
	    cod_acom_naci = x[226:232]

	    select = mycursor.execute("SELECT eleccio_id FROM eleccions")
	    fetch = mycursor.fetchone()
	    fetch = " ".join(map(str,fetch))

	    print(codi_candidatura.strip(), ",", sigles_candidatura.strip(), ",", denom_candidatura.strip(), ",", cod_acom_provi.strip(), ",", cod_acom_auto.strip(), ",", cod_acom_naci)
	    insert = ("INSERT INTO candidatures (codi_candidatura, nom_curt, nom_llarg, codi_acumulacio_provincia, codi_acumulacio_ca, codi_acumulario_nacional, eleccio_id) VALUES (%s,%s,%s,%s,%s,%s,%s)")
	    valores = [codi_candidatura, sigles_candidatura, denom_candidatura, cod_acom_provi, cod_acom_auto, cod_acom_naci, fetch]
	    mycursor.execute(insert,valores)

	mydb.commit()
	

### Importació de candidats i persones:
	Persones:
	f = open(r'C:\Users\Usurio\Desktop\02201606_MESA\04021606.DAT', 'r')
	mycursor = mydb.cursor()

	for i in f:
	    nom = i[25:49]
	    cognom = i[50:74]
	    segcognom = i[75:99]
	    dni = i[109:118]

	    sql = "INSERT INTO persones (nom,cog1,cog2,dni) VALUES (%s, %s, %s, %s)"
	    val = [nom,cognom,segcognom,dni]
	    mycursor.execute(sql, val)
	    mydb.commit()

	
	Candidats:
	f = open(r'C:\Users\Usurio\Desktop\02201606_MESA\04021606.DAT', 'r')
	mycursor = mydb.cursor(buffered=True)

	for i in archivo:
		numordre = i[21:24]
		tipus = i[24:25]
		candidaturacodi = i[15:21]
		nom = i[25:49]
		cognnom = i[50:74]
		segcognom = i[75:99]
		provinciacodi = i[9:11]	

		select = "SELECT candidatura_id FROM candidatures WHERE codi_candidatura = %s"
		dades = [candidaturacodi]
		mycursor.execute(select,dades)
		candidatura_id = mycursor.fetchone()
		candidatura_id = " ".join(map(str,candidatura_id))



		select1 = 'SELECT persona_id FROM persones WHERE nom = %s AND cog1 = %s AND cog2 = %s'
		dades1 = [nom.strip(),cognnom.strip(),segcognom.strip()]
		mycursor.execute(select1,dades1)
		persona_id = mycursor.fetchone()
		persona_id = " ".join(map(str,persona_id))



		select2 = 'SELECT provincia_id FROM provincies WHERE codi_ine = %s'
		dades2 = [provinciacodi]
		mycursor.execute(select2,dades2)
		provincia_id = mycursor.fetchone()
		provincia_id = " ".join(map(str,provincia_id))



		sql = "INSERT INTO candidats (candidatura_id,persona_id,provincia_id,num_ordre,tipus) VALUES (%s, %s, %s, %s, %s)"
		val = [candidatura_id,persona_id,provincia_id,numordre,tipus]
		mycursor.execute(sql, val)
	
	
	mydb.commit()
### Importació de la taula eleccions_municipis:

	mycursor = mydb.cursor(buffered=True)

	archivo = open(r"C:\Users\david\Desktop\Pràctica BDD\mesa 0220160\07021606.dat")

    select = mycursor.execute("SELECT eleccio_id FROM eleccions")
    fetch = mycursor.fetchone()
    fetch = " ".join(map(str,fetch))

	select1 = mycursor.execute("SELECT municipi_id FROM municipis")
    fetch1 = mycursor.fetchone()
    fetch1 = " ".join(map(str,fetch1))

    select2 = mycursor.execute("SELECT (vots_candidatures + vots_nuls) AS vots_valids FROM eleccions_municipis")
    vots_valids = mycursor.fetchone()
    vots_valids = " ".join(map(str,vots_valids))

	for x in archivo:
    	num_mesa = x[72:77]
    	cens = x[77:85]
    	vots_emesos = x[109:117]
   		vots_candidatures = x[141:149]
    	vots_blanc = x[125:133]
    	vots_nuls = x[133:141]


    	select = mycursor.execute("SELECT eleccio_id FROM eleccions")
    	fetch = mycursor.fetchone()
    	fetch = " ".join(map(str,fetch))

		select1 = mycursor.execute("SELECT municipi_id FROM municipis")
    	fetch1 = mycursor.fetchone()
    	fetch1 = " ".join(map(str,fetch1))

    	select2 = mycursor.execute("SELECT (vots_candidatures + vots_nuls) AS vots_valids FROM eleccions_municipis")
    	vots_valids = mycursor.fetchone()
    	vots_valids = " ".join(map(str,vots_valids))

    	print(fetch, ",", fetch1,",", num_mesa.strip(), ",", cens.strip(), ",", vots_emesos.strip(), ",", vots_valids.strip(), ",", vots_candidatures.strip(), ",", vots_blanc.strip(), ",", vots_nuls)


	for x in archivo:
  	# Buscar un registro existente con los mismos valores de eleccio_id y municipi_id
    	select = mycursor.execute("SELECT * FROM eleccions_municipis WHERE eleccio_id=%s AND municipi_id=%s", (fetch, fetch1))
    	existing_record = mycursor.fetchone()

    	if existing_record:
    		# Actualizar el registro existente con los valores nuevos
        		update = ("UPDATE eleccions_municipis SET num_meses=%s, cens=%s, vots_emesos=%s, vots_valids=%s, vots_candidatures=%s, vots_blanc=%s, vots_nuls=%s WHERE eleccio_id=%s AND municipi_id=%s")
       	 	valores = [num_mesa, cens, vots_emesos, vots_valids, vots_candidatures, vots_blanc, vots_nuls, fetch, fetch1]
       	 		mycursor.execute(update, valores)
    	else:
   	 		# Insertar un nuevo registro
        		insert = ("INSERT INTO eleccions_municipis (eleccio_id, municipi_id, num_meses, cens, vots_emesos, vots_valids, vots_candidatures, vots_blanc, vots_nuls) VALUES (%s,%s,%s,%s,%s,%s,%s,%s,%s)")
        		valores = [fetch, fetch1, num_mesa, cens, vots_emesos, vots_valids, vots_candidatures, vots_blanc, vots_nuls]
        		mycursor.execute(insert, valores)



	mydb.commit()
	print(mycursor.rowcount, "Se inserto correctamente!.")


### Importació de vots a nivell municipal:

	mycursor = mydb.cursor(buffered=True)
	archivo = open(r"C:\Users\david\Desktop\Pràctica BDD\mesa 0220160\06021606.dat")

	DATOS DE CANDIDATURAS DE MUNICIPIOS:

	for x in archivo:
    		vot_cand = x[22:30]
    
    		select = mycursor.execute("SELECT eleccio_id FROM eleccions")
    		eleccio_id = mycursor.fetchone()
    		eleccio_id = " ".join(map(str,eleccio_id))

    		select1 = mycursor.execute("SELECT municipi_id FROM municipis")
    		municipi_id = mycursor.fetchone()
		municipi_id = " ".join(map(str,municipi_id))

    		select2 = mycursor.execute("SELECT candidatura_id FROM candidatures")
    		candidatura_id = mycursor.fetchone()
    		candidatura_id = " ".join(map(str,candidatura_id))

   		print(eleccio_id, ",", municipi_id, ",", candidatura_id, ",", vot_cand)
    
    		insert = ("INSERT INTO vots_candidatures_mun (eleccio_id, municipi_id, candidatura_id, vots) VALUES (%s,%s,%s,%s)")
    		valores = [eleccio_id, municipi_id, candidatura_id, vot_cand]
    
   		mycursor.execute(insert,valores)

	mydb.commit()


### Importació de vots a nivell provincial:

	mycursor = mydb.cursor(buffered=True)
	
	archivo = open(r"C:\Users\david\Desktop\Pràctica BDD\mesa 0220160\08021606.dat")

	DATOS DE CANDIDATURAS DE PROVINCIES:

	for x in archivo:
    		vot_prov = x[20:28]
    		cand_obtin = x[28:33]
    
    		select3 = mycursor.execute("SELECT provincia_id FROM provincies")
    		provincia_id = mycursor.fetchone()
		provincia_id = " ".join(map(str,provincia_id))


    		select4 = mycursor.execute("SELECT candidatura_id FROM candidatures")
    		candidatura_id = mycursor.fetchone()
    		candidatura_id = " ".join(map(str,candidatura_id))

    		print(provincia_id, ",", candidatura_id, ",", vot_prov, ",", cand_obtin)
    
    		insert = ("INSERT INTO vots_candidatures_prov (provincia_id, candidatura_id, vots, candidats_obtinguts) VALUES (%s,%s,%s,%s)")
    		valores = [provincia_id, candidatura_id, vot_prov, cand_obtin]
    
    		mycursor.execute(insert,valores)

	mydb.commit()


### Importació de vots a nivell autonòmic: 

	mycursor = mydb.cursor(buffered=True)

	archivo = open(r"C:\Users\david\Desktop\Pràctica BDD\mesa 0220160\08021606.dat")

	DATOS DE CANDIDATURAS DE COMUNITATS AUTONOMES:

	for x in archivo:
    		vot_ca = x[20:28]
    		comunitat_autonoma_id = x[9:11]

    		select5 = mycursor.execute("SELECT candidatura_id FROM candidatures")
    		candidatura_id = mycursor.fetchone()
    		candidatura_id = " ".join(map(str,candidatura_id))


    		print(comunitat_autonoma_id, ",", candidatura_id, ",", vot_ca)
    
    		insert = ("INSERT INTO vots_candidatures_ca (comunitat_autonoma_id, candidatura_id, vots) VALUES (%s,%s,%s)")
    		valores = [comunitat_autonoma_id, candidatura_id, vot_ca]
    
    		mycursor.execute(insert,valores)

	mydb.commit()


## APARTAT 2: Creació de sentències de consulta SQL:

### Categoria 1: 5 preguntes de consultes simples: inclou una sola taula, funcions, funcions d'agregat o grups:

##### Mostrar candidatura_id, codi_candidatura, nom_curt, nom_llarg de les candidatures les quals en el seu nom contingui partido socialista i ordena per candidatura_id.

SELECT candidatura_id, codi_candidatura, nom_curt, nom_llarg
	FROM candidatures
WHERE nom_llarg LIKE '%partido socialista%'
ORDER BY candidatura_id;

/*-------------------------------------------------------------------------*/

##### Mostrar el codi INE d'Extremadura.

SELECT codi_ine AS codi_INE_extremadura
	FROM comunitats_autonomes
WHERE nom = 'extremadura';

/*-------------------------------------------------------------------------*/

##### Mostrar candidat_id, persona_id, provincia_id dels candidats que el seu ID estigui entre 6 i 15.

SELECT candidat_id, persona_id, provincia_id
	FROM candidats
WHERE candidat_id BETWEEN 6 AND 15;

/*-------------------------------------------------------------------------*/

##### Mostrar persona_id, cog1, cog2, nom de les persones que tinguin com a primer cognom Martínez o Ruiz i ordenar-lo per ID.

SELECT persona_id, cog1, cog2, nom
	FROM persones
WHERE cog1 = 'Martínez' OR cog1 = 'Ruiz'
ORDER BY persona_id;

/*-------------------------------------------------------------------------*/

##### Mostrar  provincia_id, nom, codi_ine de les provincies que en el seu nom continguin 'illa'.

SELECT provincia_id, nom, codi_ine
	FROM provincies
WHERE nom LIKE '%illa%';



### Categoria 2: 5 preguntes de consultes de combinacions de més d'una taula: INNER JOINS, LEFT JOINS: 

##### Quines provincies pertanyen a la comunitat autonoma amb el codi INE(08)?

SELECT p.nom ,c.codi_ine
	FROM provincies p
INNER JOIN comunitats_autonomes c ON p.comunitat_aut_id = c.comunitat_aut_id
WHERE c.codi_ine = '08';

##### Quin és el numero de municipis de cada provincia d'Andalucia?

SELECT p.nom AS provincia, COUNT(m.municipi_id) AS num_municipis
FROM provincies p
LEFT JOIN municipis m ON p.provincia_id = m.provincia_id
LEFT JOIN comunitats_autonomes c ON p.comunitat_aut_id = c.comunitat_aut_id
WHERE c.nom = 'Andalucía'
GROUP BY p.provincia_id;

### Categoria 3: 5 preguntes fent ús de subconsultes:



### Categoria 4: 1 pregunta utilitzant WINDOW FUNCTIONS o recursivitat:
  
 

## APARTAT 4: Llei d'hondt:

Implementa un programa que mostri el nº d’escons per una província passada per paràmetre.
