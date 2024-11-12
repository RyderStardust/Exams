## Programmation Réseau en Go - Détails Supplémentaires

### 1. Protocoles et Adressage
Les protocoles réseau comme **TCP** (Transmission Control Protocol) et **UDP** (User Datagram Protocol) sont essentiels. Go permet de les implémenter via le package `net`.

   - **TCP** est un protocole orienté connexion : il garantit que les données arrivent intactes et dans le bon ordre, mais il est plus lent.
   - **UDP** est un protocole sans connexion : il est plus rapide, mais les paquets peuvent se perdre ou arriver dans le désordre.

### 2. Connexions UDP
Pour les connexions **UDP**, le processus est similaire à TCP, mais on n’a pas besoin d’établir de connexion formelle (via un `Dial`). Cela peut être utile pour des applications nécessitant de la vitesse, comme les jeux vidéo ou les transmissions de données en direct.

#### Serveur UDP
   ```go
   package main

   import (
       "fmt"
       "net"
   )

   func main() {
       addr := net.UDPAddr{
           Port: 8081,
           IP:   net.ParseIP("127.0.0.1"),
       }
       conn, _ := net.ListenUDP("udp", &addr)
       defer conn.Close()

       buffer := make([]byte, 1024)
       for {
           n, addr, _ := conn.ReadFromUDP(buffer)  // Lecture des données et de l'adresse source
           fmt.Printf("Reçu de %s : %s\n", addr, string(buffer[:n]))
       }
   }
   ```

#### Client UDP
   ```go
   package main

   import (
       "fmt"
       "net"
   )

   func main() {
       serverAddr := net.UDPAddr{
           Port: 8081,
           IP:   net.ParseIP("127.0.0.1"),
       }
       conn, _ := net.DialUDP("udp", nil, &serverAddr)
       defer conn.Close()

       conn.Write([]byte("Hello UDP Server"))  // Envoi de données au serveur
   }
   ```

### 3. Utilisation des JSON pour la Transmission de Données
Pour échanger des données structurées entre un client et un serveur, on peut utiliser le format JSON. Go fournit le package `encoding/json` pour sérialiser et désérialiser des données en JSON.

   ```go
   package main

   import (
       "encoding/json"
       "fmt"
   )

   type Message struct {
       Sender  string `json:"sender"`
       Content string `json:"content"`
   }

   func main() {
       msg := Message{Sender: "Client", Content: "Hello, Server!"}
       data, _ := json.Marshal(msg)          // Convertit en JSON
       fmt.Println(string(data))             // Affiche le JSON sérialisé

       var decoded Message
       json.Unmarshal(data, &decoded)        // Reconvertit en structure Go
       fmt.Println(decoded.Sender, decoded.Content)
   }
   ```

---

## Concurrence avec Goroutines et Channels - Plus de Détails

### Goroutines : Gestion de Concurrence Massivement Parallèle
Les goroutines sont extrêmement légères en comparaison des threads systèmes, permettant de gérer des milliers de tâches en parallèle sans surcharger la mémoire.

   - **Lancement d’une goroutine** : Utilise `go` devant l’appel de fonction.
   - **Synchronisation** : Go ne possède pas de verrous de thread traditionnels ; il préfère les channels pour partager des données entre goroutines.

### Channels - Transmission de Données Entre Goroutines
Les channels peuvent être **bloquants** ou **non-bloquants**, et peuvent être fermés pour signaler la fin de transmission.

#### Exemple Avancé : Synchronisation des Goroutines avec un Channel
   ```go
   package main

   import (
       "fmt"
       "time"
   )

   func worker(done chan bool) {
       fmt.Println("Travail en cours...")
       time.Sleep(2 * time.Second)
       fmt.Println("Travail terminé")
       done <- true // Signal de fin
   }

   func main() {
       done := make(chan bool)
       go worker(done)  // Lancement de worker dans une goroutine

       <-done           // Attente de la fin de la goroutine worker
       fmt.Println("Fin du programme")
   }
   ```

### Channels Buffered et Unbuffered
   - **Channels unbuffered** : Ils bloquent l’envoi de données jusqu’à ce qu’un récepteur soit prêt.
   - **Channels buffered** : Ils stockent une certaine quantité de données, permettant de traiter plusieurs éléments sans bloquer immédiatement.
     ```go
     ch := make(chan int, 2) // Channel avec une capacité de 2
     ```

---

## Raft - Un Algorithme de Consensus Approfondi

### Fonctionnement des États dans Raft
Chaque nœud dans Raft peut se trouver dans un de ces trois états :
   - **Follower** : Accepte les requêtes du leader.
   - **Candidate** : Se déclare candidat lors d’une élection.
   - **Leader** : Responsable de l’ajout de nouvelles entrées au journal.

### Détails des Élections
Lorsqu’un follower ne reçoit pas de signal (ou « heartbeat ») du leader, il se met en état de **candidate** et demande aux autres nœuds de voter pour lui. Si une majorité de nœuds lui accorde leur vote, il devient le nouveau **leader**.

#### Pseudocode d’une Élection Raft
   ```pseudo
   for each node:
       if timeout:
           start election
           send vote request to other nodes
           if receive majority votes:
               become leader
   ```

### Journal et Réplication de Log
Le leader transmet les **entrées de journal** aux followers pour maintenir la cohérence des données. Une entrée de journal est validée lorsque la majorité des nœuds l'ont enregistrée.

---

## DHT - Tables de Hachage Distribuées en Profondeur

### Fonctionnement des Tables de Hachage Distribuées (DHT)
Les DHT répartissent les données entre des nœuds en utilisant une fonction de hachage, qui mappe chaque **clé** sur un **nœud responsable**.

   - **Fonction de Hachage** : Elle convertit une clé (souvent un nom de fichier ou une autre chaîne de caractères) en un identifiant numérique qui correspond à un nœud dans le réseau.
   - **Partitionnement** : La plage des valeurs de hachage est répartie entre les nœuds pour équilibrer la charge.

### Algorithme Consistent Hashing
Le **consistent hashing** est une méthode de hachage qui permet de répartir les clés entre les nœuds même en cas d’ajout ou de suppression de nœuds, avec un minimum de modifications sur la répartition des données.

   - Chaque nœud et clé a un identifiant basé sur une fonction de hachage.
   - Les clés sont attribuées aux nœuds en trouvant le premier nœud ayant un identifiant supérieur ou égal à celui de la clé.

#### Exemple de Hachage Consistent Simplifié en Go
   ```go
   package main

   import (
       "fmt"
       "hash/fnv"
   )

   type Node struct {
       id    int
       data  map[string]string
   }

   func hashKey(key string) int {
       h := fnv.New32a()
       h.Write([]byte(key))
       return int(h.Sum32() % 256) // Clé dans la plage de 0 à 255
   }

   func storeData(nodes []Node, key, value string) {
       hash := hashKey(key)
       for _, node := range nodes {
           if node.id >= hash {
               node.data[key] = value
               fmt.Printf("Clé %s stockée sur le nœud %d\n", key, node.id)
               return
           }
       }
       nodes[0].data[key] = value // Si le hash est plus grand que tous les ids
   }

   func main() {
       nodes := []Node{
           {id: 50, data: make(map[string]string)},
           {id: 100, data: make(map[string]string)},
           {id: 200, data: make(map[string]string)},
       }

       storeData(nodes, "filename1", "some data")
       storeData(nodes, "filename2", "other data")
   }
   ```

   - **Avantage** : Permet d’équilibrer les données et de faciliter l'ajout ou la suppression de nœuds sans déplacer toutes les données.

---
