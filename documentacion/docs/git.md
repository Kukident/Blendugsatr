#Comandos basicos
```bash
git status							;Estado de git
git clone {urlrepositorio}
git checkout -b {nombrebranch}		;Crea una nueva branch
git push -u origin {nombrebranch}	;Sube a github la branch creada
git checkout {nombrebranch}			;Cambia a la branch indicada (No pierdes los cambios hechos en los ficheros)
git checkout {fichero}				;Deshace los cambios realizados al fichero
git add {fichero}					;Añade ficheros a la lista de ficheros que se subiran a github
git commit -m "{mensaje}"			;Añade los ficheros modificados a un commit
git push							;Sube los commits hechos a github
git pull							;Descarga los cambios hechos en la branch actual desde el repositorio de github
```


Notas:

* Una vez hecho un commit en github es una rallada quitarlo, asi que mejor no la lieis. En local no hay fallo, podeis hacer y deshacer todo lo que querais, mientras que no lo commiteis.
* Siempre es recomendable hacer una branch al trabajar en una nueva funcionalidad, una vez terminada se hace un pullrequest para juntarlo con la branch principal.
* Tambien es recomendable hacer git pull para traernos los cambios realizados.
* Para ver que es lo que se va commitear usar el comando git status y mucho ojo en que branch haceis el commit