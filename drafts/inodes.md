Il discorso sui blocchi a mio avviso è leggermente diverso: i blocchi vengono allocati solamente per i dati associati ai file; non avendo dati associati, la directory usa zero blocchi ma la sua dimensione all'interno della struttura dati che contiene gli inode cambia in base al suo contenuto. Ad esempio sul mio computer, eseguendo questo one-liner:

DIR=~/Downloads; # imposto la variabile DIR al nome della cartella su cui voglio eseguire i comandi \ 
stat $DIR; # chiamo stat sulla cartella \
printf "\n Number of files in directory: "; # mi stampo un messaggio di info \
find $DIR -maxdepth 1 | wc -l # conto quante entry ha la cartella

Ottengo i seguenti risultati:

$ DIR=~/Downloads; stat $DIR; printf "\n Number of files in directory: "; find $DIR -maxdepth 1 | wc -l;
  File: /home/jmalta/Downloads
  Size: 2142       Blocks: 0          IO Block: 4096   directory
Device: 0,42 Inode: 284         Links: 1
Access: (0755/drwxr-xr-x)  Uid: ( 1000/  jmalta)   Gid: ( 1000/  jmalta)
Context: unconfined_u:object_r:user_home_t:s0
Access: 2024-10-31 12:03:04.110877538 +0100
Modify: 2024-10-31 12:02:33.316942513 +0100
Change: 2024-10-31 12:02:33.316942513 +0100
 Birth: 2024-07-01 11:11:16.597065648 +0200

 Number of files in directory: 43
$ DIR=/etc; stat $DIR; printf "\n Number of files in directory: "; find $DIR -maxdepth 1 | wc -l;
  File: /etc
  Size: 4656       Blocks: 0          IO Block: 4096   directory
Device: 0,35 Inode: 270         Links: 1
Access: (0755/drwxr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
Context: system_u:object_r:etc_t:s0
Access: 2024-10-08 17:59:26.930164525 +0200
Modify: 2024-10-29 12:17:34.880130368 +0100
Change: 2024-10-29 12:17:34.880130368 +0100
 Birth: 2024-07-01 10:58:53.103405472 +0200

 Number of files in directory: 275
$ DIR=/; stat $DIR; printf "\n Number of files in directory: "; find $DIR -maxdepth 1 | wc -l;
  File: /
  Size: 158       Blocks: 0          IO Block: 4096   directory
Device: 0,35 Inode: 256         Links: 1
Access: (0555/dr-xr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
Context: system_u:object_r:root_t:s0
Access: 2024-10-30 15:12:55.002721257 +0100
Modify: 2024-04-15 00:57:12.460653901 +0200
Change: 2024-07-01 11:02:30.159997199 +0200
 Birth: 2024-07-01 10:58:47.576434706 +0200

 Number of files in directory: 22

Vediamo che il numero di blocchi associati alla cartella è sempre zero (l'IO Block non c'entra nulla, qui trova una spiegazione abbastanza chiara https://askubuntu.com/questions/946521/what-is-the-meaning-of-io-block-and-how-is-it-calculated) ma la dimensione cambia in base al numero di file contenuti nella cartella.
