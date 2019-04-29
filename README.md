L’objectiu d’aquesta activitat és realitzar una sèrie de funcions realitzades sobre el fitxer
.sql adjunt a la pràctica. El nom del fitxer és practica.sql ( ja ho heu vist a l’anterior pràctica)

No s’acceptaran entregues fora de la data establerta sí no és amb justificació mèdica
o qüestions de força major.

## 1 – Dissenya les següents funcions.

1) Quin serà el resultat obtingut desprès d’executar la funció Exemple ?

```plsql
Create function Exemple() returns Integer as $$
Declare
	Valor integer := 50;
Begin
	Raise Notice 'Valor val %', Valor;
	Valor := 55;
	--
	-- Creem un sub-bloc.
	--
		Declare
			Valor Integer := 21;
		Begin
			Raise Notice 'Valor dins del sub-bloc val % ', Valor ;
		End;
	Raise Notice 'Valor val %',Valor;
	Return Valor ;
End;
$$ Language plpgsql;
```

> El valor obtingut sera: 55
>
> I treura els missatges:
>
> ​	NOTICE:  Valor val 50
> ​	NOTICE:  Valor dins del sub-bloc val 21
> ​	NOTICE:  Valor val 55

2) Creeu una taula, anomenada RegEntrada, amb els següents camps:

| Nom del camp | Tipus                   | Descripció                        |
| ------------ | ----------------------- | --------------------------------- |
| Id           | Serial, Clau principal. | Serà l’identificador de la taula. |
| Data         | TimeStamp               | Contindrà la data d’alta.         |
| Nom          | Varchar(50).            | Contindrà una descripció.         |

```plsql
CREATE TABLE RegEntrada(
	Id serial PRIMARY KEY,
	Data TimeStamp,
	Nom Varchar(50)
);
```

3) Teclejeu les dues funcions següents i responeu les preguntes:
Nota: busqueu la funció now en la vostra versió de Postgres que retorna la data actual.

```plsql
Create Function Test1(text) Returns TimeStamp As $$
Declare
	texte Alias For $1;
Begin
	Insert into RegEntrada (data,nom) values( 'now', texte);
	Return 'now';
End;
$$ Language plpgsql;
```

```plsql
Create Function Test2(text) Returns TimeStamp As $$
Declare
	texte Alias For $1;
	dataActual timeStamp;
Begin
	DataActual := 'now';
	Insert into RegEntrada (data,nom) values( 'now', dataActual);
	Return dataActual;
End;
$$ Language plpgsql;
```

a ) Que fan aquestes funcions ?

> Les dues funcions executen un insert dins de la taula RegEntrada

b ) Funcionen les dues igual ? En cas contrari on esta la diferència i perquè ?

> Si i no:
>
> Les dues funcions fan un insert a la taula, pero la primera funcio 'Test1' inserte una fila amb la fecha i hora actual al camp Data i al camp Nom el text que li has passat a la funcio, i la funcio 'Test2' encanvi inserte tant al camp Nom com al camp Data echa i hora actual.
>
> PD: Les dues funcions retornen la data i hora actual.

**A partir d’aqui necessitareu el fitxer practica.sql .**

4) Crear una funció que retorni cert si existeix l’assignatura amb idassignatura igual a 3 i fals
altrament.

```plsql
CREATE OR REPLACE FUNCTION existeixAssignatura() RETURNS BOOLEAN As $$
BEGIN
	IF (SELECT COUNT(idassignatura) FROM Assignatura WHERE idassignatura=3) = 1 THEN
		RETURN TRUE;
	ELSE
		RETURN FALSE;
	END IF;
END;
$$ LANGUAGE plpgsql;

SELECT existeixAssignatura();
```

5) Modificar la funció anterior per tal que puguem passar com a paràmetre el codi de
l’assignatura a buscar.

```plsql
CREATE OR REPLACE FUNCTION existeixAssignatura(codiAssignatura varchar(20)) RETURNS BOOLEAN As $$
BEGIN
	IF (SELECT COUNT(idassignatura) FROM Assignatura WHERE idassignatura=3 AND nom=codiAssignatura ) = 1 THEN
		RETURN TRUE;
	ELSE
		RETURN FALSE;
	END IF;
END;
$$ LANGUAGE plpgsql;

SELECT existeixAssignatura('ESOFT');
```

6) Crear una funció que retorni la suma de les notes de tots els alumnes de ‘Lleida’.

```plsql
CREATE OR REPLACE FUNCTION sumaNotes() RETURNS INT As $$
BEGIN
	RETURN (SELECT SUM(nota) 
	FROM Notes AS N INNER JOIN Alumne AS A ON N.idalumne = A.idalumne
	WHERE A.ciutat LIKE 'Lleida');
END;
$$ LANGUAGE plpgsql;

SELECT sumaNotes();
```

7) Crear una funció que doni d’alta un Alumne. Cal passar-li com a paràmetres tots els camps
menys l’identificador.

```plsql
CREATE OR REPLACE FUNCTION insertAlumne(nom varchar(20),edat int,ciutat varchar(20)) RETURNS VARCHAR As $$
BEGIN
	INSERT INTO Alumne (Nom,Edat,Ciutat) VALUES (nom,edat,ciutat);

	RETURN 'Insert executat correctament';
END;
$$ LANGUAGE plpgsql;

SELECT insertAlumne('Dani',24,'Balaguer');
```

8)Crear una funció que doni de baixa un Alumne donat. (passant-li com a paràmetre
l’idalumne).

```plsql
CREATE OR REPLACE FUNCTION deleteAlumne(id int) RETURNS VARCHAR As $$
BEGIN
	DELETE FROM Alumne WHERE idalumne = id;

	RETURN 'Delete executat correctament';
END;
$$ LANGUAGE plpgsql;
```

9) Crear una funció que esborri una nota donada passant només l’idalumne i l’idassignatura i
que ens retorni la nota que hi havia a la taula notes. En cas de no existir aquesta nota ficada
cal retornar –1.

```plsql
CREATE OR REPLACE FUNCTION deleteNota(idAlum int,idAss int) RETURNS int As $$
DECLARE
	notaR INTEGER = null;
BEGIN
	IF(SELECT COUNT(*) FROM Notes WHERE idalumne = idAlum AND idassignatura = idAss) = 1 THEN
		SELECT nota INTO notaR FROM Notes WHERE idalumne = idAlum AND idassignatura = idAss;
		DELETE FROM Notes WHERE idalumne = idAlum AND idassignatura = idAss;
		RETURN notaR;
	ELSE
		RETURN -1;
	END IF;
END;
$$ LANGUAGE plpgsql;

SELECT deleteNota(1,1);
```

10) Crear una funció que doni d’alta a la taula Notes. Aquesta funció només tindrà els 3
paràmetres d’aquesta taula. En cas de que el codi de l’alumne no existeixi cal donar d’alta un
nou Alumne, amb valors per defecte (Nom = ‘Ferran’, Edat = 21, Ciutat = ‘Lleida’), i assignar
a la taula notes el nou codi de l’alumne assignat. Fer el mateix pel cas de la taula Assignatura
(Nom = ‘Clau’, Numalumnes = 10).

```plsql
CREATE OR REPLACE FUNCTION insertNota(idAlum int,idAss int,nota int) RETURNS int As $$
DECLARE
	retorn INT = 0;
	codiAl INT = -1;
	codiAss INT = -1;
BEGIN

	IF((SELECT COUNT(*) FROM Alumne WHERE idalumne = idAlum) = 1) AND ((SELECT COUNT(*) FROM Assignatura WHERE idassignatura = idAss) = 1) THEN
		INSERT INTO Notes(idalumne,idassignatura,nota) VALUES (idAlum,idAss,nota);
		retorn := 0;
	ELSE
		IF(SELECT COUNT(*) FROM Alumne WHERE idalumne = idAlum) != 1 THEN
			INSERT INTO Alumne (Nom,Edat,Ciutat) VALUES ('Ferran',21,'Lleida');
			SELECT idalumne INTO codiAl FROM Alumne WHERE Nom LIKE 'Ferran' AND Edat = 21 AND Ciutat LIKE 'Lleida';
			retorn := retorn + 1;
		END IF;

		IF(SELECT COUNT(*) FROM Assignatura WHERE idassignatura = idAss) != 1 THEN
			INSERT INTO Assignatura (Nom,NumAlumnes) VALUES ('Clau',10);
			SELECT idassignatura INTO codiAss FROM Assignatura WHERE nom LIKE 'Clau' AND numalumnes = 10;
			retorn := retorn + 10;
		END IF;

		IF(codiAl != -1 AND codiAss != -1)THEN
			INSERT INTO Notes(idalumne,idassignatura,nota) VALUES (codiAl,codiAss,nota);
		ELSE
			IF codiAl != -1 THEN
				INSERT INTO Notes(idalumne,idassignatura,nota) VALUES (codiAl,idAss,nota);
			END IF;

			IF codiAss != -1 THEN
				INSERT INTO Notes(idalumne,idassignatura,nota) VALUES (idAlum,codiAss,nota);
			END IF;
		END IF;
	END IF;
	RETURN retorn;
	
END;
$$ LANGUAGE plpgsql;

SELECT insertNota(100,100,2);
```

La funció retornarà un codi d’estatus que valdrà:

| Codi estatus | Descripció                                                   |
| ------------ | ------------------------------------------------------------ |
| 0            | S’ha fet l’alta normalment                                   |
| 1            | S’ha donat d’alta un nou alumne. I s’ha utilitzat el codi d’aquest per la taula notes |
| 10           | S’ha donat d’alta una nova assignatura. I s’ha utilitzat el codi d’aquest per la taula notes. |
| 11           | S’ha donat d’alta un alumne i una assignatura. I s’han utilitzat ambdós codis per la taula notes. |

Ajuda:
• Sentencies Bàsiques. Select Into:

• La sentència [NOT] FOUND permet saber si el resultat d’una sentencia SELECT INTO
ha tingut èxit o no.

```plsql
SELECT INTO variableDesti expressions From .... ;
Exemple:
	Declare
		Prove Proveidor%ROWTYPE;
	Begin
		Select Into Prove * From Proveidor Wherr IdProveidor=2;
```

• La sentència [NOT] FOUND permet saber si el resultat d’una sentencia SELECT INTO
ha tingut èxit o no.
