## Programmation Réseau en Go - Concepts Avancés

### Gestion des Erreurs dans les Connexions Réseau
Une bonne gestion des erreurs est cruciale en programmation réseau, car les connexions peuvent échouer pour diverses raisons (problèmes de réseau, délais d'attente, etc.). En Go, les erreurs sont souvent renvoyées sous forme de valeurs `error` qu’il est important de gérer.

   ```go
   conn, err := net.Dial("tcp", "127.0.0.1:8080")
   if err != nil {
       log.Fatalf("Erreur lors de la connexion : %s", err)
   }
   defer conn.Close()
   ```

   - Utiliser `log.Fatalf` pour des erreurs graves qui arrêtent le programme.
   - Pour des erreurs moins graves, il est souvent plus sûr d’afficher un message et de permettre au programme de continuer ou de réessayer.

### Timeout et Connexions Non-Bloquantes
Les **timeouts** permettent de limiter le temps d’attente pour une connexion ou une opération, ce qui évite que le programme ne bloque indéfiniment.

   ```go
   conn, err := net.DialTimeout("tcp", "127.0.0.1:8080", time.Second*5)
   if err != nil {
       log.Println("Connexion expirée")
       return
   }
   defer conn.Close()
   ```

   Pour les connexions réseau longues, il est souvent nécessaire d’ajouter des **timeouts** pour éviter de bloquer le programme si le serveur ne répond pas.

### Programmation Orientée Paquets
Certains protocoles, comme UDP, fonctionnent avec des paquets indépendants. Go permet d'envoyer et de recevoir des paquets individuellement, ce qui est idéal pour des systèmes temps réel.

   ```go
   conn, _ := net.ListenPacket("udp", ":8080")
   defer conn.Close()

   buffer := make([]byte, 1024)
   n, addr, _ := conn.ReadFrom(buffer) // Lire un paquet UDP
   fmt.Printf("Reçu %d octets de %s : %s\n", n, addr, string(buffer[:n]))
   ```

---

## Concurrence Avancée avec Goroutines et Channels

### Sélection dans les Channels (`select`)
La clause `select` permet de gérer plusieurs channels en parallèle et d’attendre qu’au moins un d’entre eux soit prêt. C’est un concept central pour les systèmes concurrents en Go.

   ```go
   package main

   import (
       "fmt"
       "time"
   )

   func main() {
       ch1 := make(chan string)
       ch2 := make(chan string)

       go func() {
           time.Sleep(2 * time.Second)
           ch1 <- "Message du channel 1"
       }()

       go func() {
           time.Sleep(1 * time.Second)
           ch2 <- "Message du channel 2"
       }()

       select {
       case msg1 := <-ch1:
           fmt.Println(msg1)
       case msg2 := <-ch2:
           fmt.Println(msg2)
       case <-time.After(3 * time.Second):
           fmt.Println("Aucun message reçu")
       }
   }
   ```

   Dans cet exemple :
   - **`select`** surveille les deux channels.
   - **`time.After`** agit comme une minuterie. Si aucun message n’est reçu dans le temps imparti, il déclenche l’affichage du message "Aucun message reçu".

### Workers Pools
Les **pools de workers** (ou **pools de goroutines**) sont une technique pour limiter le nombre de goroutines simultanées. Ils sont particulièrement utiles pour les tâches de traitement de masse où l'on veut un nombre fixe de workers.

   ```go
   package main

   import (
       "fmt"
       "sync"
   )

   func worker(id int, jobs <-chan int, results chan<- int) {
       for job := range jobs {
           fmt.Printf("Worker %d traite job %d\n", id, job)
           results <- job * 2
       }
   }

   func main() {
       jobs := make(chan int, 5)
       results := make(chan int, 5)

       var wg sync.WaitGroup
       for w := 1; w <= 3; w++ {
           wg.Add(1)
           go func(id int) {
               defer wg.Done()
               worker(id, jobs, results)
           }(w)
       }

       for j := 1; j <= 5; j++ {
           jobs <- j
       }
       close(jobs)

       wg.Wait() // Attendre que tous les workers finissent
       close(results)
   }
   ```

   Ici :
   - **3 workers** sont créés et traitent les jobs en parallèle.
   - Les résultats sont stockés dans le channel `results`.
   - **`sync.WaitGroup`** attend que tous les workers aient terminé avant de fermer le programme.

---

## Algorithme de Consensus Raft - Détails Approfondis

### Architecture d’un Cluster Raft
Un cluster Raft typique se compose de **3 à 7 nœuds** pour une redondance et une disponibilité optimales. Chaque nœud peut être dans un état de **Leader**, **Candidate**, ou **Follower**. 

   - **Election** : Lorsqu’un nœud ne reçoit pas de message `heartbeat` d’un leader, il passe en mode **Candidate** et tente d’obtenir une majorité de votes pour devenir le leader.
   - **Append Entries** : Le leader envoie périodiquement des messages `Append Entries` pour synchroniser les journaux des followers.

### Gestion des Logs dans Raft
Les entrées de journal contiennent des **commandes** (par exemple, des modifications d'état). Lorsqu'un leader reçoit une requête de mise à jour, il ajoute la commande à son journal et la transmet aux followers pour assurer la cohérence.

   - **Validation** : Une entrée est considérée comme validée si la majorité des nœuds l’ont confirmée.
   - **Propagation** : Le leader s’assure que tous les followers reçoivent la même version du journal pour maintenir la cohérence.

#### Pseudocode de Synchronisation des Journaux
   ```pseudo
   for each entry in log:
       send entry to followers
       if majority of followers confirm:
           commit entry to log
   ```

### Sécurité et Termes dans Raft
   - **Termes** : Un terme est une unité de temps pendant laquelle un leader reste en fonction. À chaque élection, le terme augmente.
   - **Logs Conflicting** : Si un follower a un log avec des entrées contradictoires (par rapport au leader), Raft corrige en lui envoyant les nouvelles entrées à partir du point de divergence.

---

## Tables de Hachage Distribuées (DHT) - Explications Avancées

### Protocole Kademlia
Kademlia est l'un des algorithmes DHT les plus utilisés pour organiser les nœuds dans un réseau. Il repose sur la **distance XOR** pour déterminer quels nœuds sont les plus proches d’une clé spécifique.

   - **ID des Nœuds** : Les nœuds et clés ont des identifiants uniques obtenus via une fonction de hachage.
   - **Routage** : Chaque nœud conserve une table de routage qui lui permet de savoir quels autres nœuds sont proches de lui. Lorsqu'un nœud doit trouver une clé, il interroge les nœuds proches pour réduire l’espace de recherche.

#### Calcul de Distance XOR
   ```go
   func xorDistance(a, b []byte) int {
       distance := 0
       for i := range a {
           distance += int(a[i] ^ b[i])  // XOR bit à bit
       }
       return distance
   }
   ```

### Opérations Clés dans un DHT
1. **Stockage d’un Objet** : Un objet est stocké sur le nœud dont l’identifiant est le plus proche du hachage de la clé.
2. **Recherche d’une Clé** : Pour rechercher une clé, on interroge d’abord les nœuds proches et ceux-ci redirigent la requête vers des nœuds encore plus proches jusqu’à trouver la clé.

---
