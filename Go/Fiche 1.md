### 1. **Introduction à Go (Golang)**
   - **Syntaxe** : Go est connu pour sa syntaxe simple et lisible, proche de C.
   - **Types de données** : Go est typé statiquement, ce qui veut dire que chaque variable a un type précis.
     - Exemples : `int`, `float64`, `string`, `bool`, `[]int` (tableau d'entiers), etc.
   - **Déclaration de variables** :
     ```go
     var x int = 10 // déclaration avec type
     y := 20 // déclaration avec inférence de type
     ```
   - **Structure de base d'un programme Go** :
     ```go
     package main

     import "fmt"

     func main() {
         fmt.Println("Hello, Go!")
     }
     ```

### 2. **Programmation réseau avec Go**
   - **Sockets** : Go simplifie la gestion des connexions réseau avec les packages `net` et `net/http`.
   - **Connexion TCP** :
     - Serveur :
       ```go
       package main

       import (
           "fmt"
           "net"
       )

       func main() {
           listener, _ := net.Listen("tcp", ":8080")
           defer listener.Close()
           for {
               conn, _ := listener.Accept()
               go handleConnection(conn)
           }
       }

       func handleConnection(conn net.Conn) {
           fmt.Fprintln(conn, "Hello from server")
           conn.Close()
       }
       ```
     - Client :
       ```go
       package main

       import (
           "fmt"
           "net"
       )

       func main() {
           conn, _ := net.Dial("tcp", "localhost:8080")
           defer conn.Close()
           buffer := make([]byte, 1024)
           conn.Read(buffer)
           fmt.Println(string(buffer))
       }
       ```

### 3. **Goroutines et Concurrence**
   - **Goroutines** : Les goroutines sont des threads légers pour l'exécution concurrente.
     ```go
     func sayHello() {
         fmt.Println("Hello")
     }

     func main() {
         go sayHello() // Lance sayHello en tant que goroutine
         fmt.Println("World")
     }
     ```
   - **Channels** : Les channels permettent de communiquer entre les goroutines.
     ```go
     func main() {
         ch := make(chan int)
         go func() {
             ch <- 10 // Envoie 10 dans le channel
         }()
         fmt.Println(<-ch) // Reçoit la valeur depuis le channel
     }
     ```

---

### 1. **Programmation Distribuée**
   - **Définition** : La programmation distribuée implique plusieurs processus qui communiquent et collaborent sur plusieurs machines pour atteindre un objectif commun.
   - **Caractéristiques des systèmes distribués** :
     - **Répartition des tâches** : Le travail est divisé entre plusieurs nœuds.
     - **Tolérance aux pannes** : Si un nœud échoue, le système peut continuer à fonctionner.
     - **Concurrence et Consistance** : La difficulté principale est de maintenir la cohérence des données, même quand plusieurs nœuds y accèdent en parallèle.
   - **Goroutines et Channels en Go** : Go simplifie l'implémentation des systèmes distribués grâce aux goroutines (qui permettent l’exécution concurrente) et aux channels (pour la communication entre les goroutines).

---

### 2. **Algorithme de Consensus : Raft**
   - **But de Raft** : Raft est un algorithme de consensus conçu pour que plusieurs nœuds (serveurs) prennent des décisions de manière coordonnée et cohérente, même en cas de pannes. Raft permet donc de maintenir un journal de commandes cohérent entre les serveurs.
   - **Terminologie et Concepts-clés de Raft** :
     - **Leader, Follower et Candidate** :
       - **Leader** : Un seul nœud est le leader et prend les décisions pour le cluster.
       - **Follower** : Les nœuds qui suivent le leader et acceptent ses ordres.
       - **Candidate** : Si un leader échoue, un nœud follower peut se proposer en tant que candidate pour devenir leader.
     - **Élections** :
       - Si un follower ne reçoit pas de message du leader pendant un certain temps (timeout), il devient candidate et initie une élection.
       - Pour être élu leader, un candidate doit obtenir la majorité des votes des autres nœuds.
     - **Log Replication (Réplication du journal)** :
       - Le leader envoie des entrées de journal aux followers, qui répliquent ces entrées dans leur propre journal pour maintenir la cohérence des données.
   - **Étapes de Raft** :
     1. **Élection du Leader** : Si le leader tombe en panne, une élection est lancée, et un nouveau leader est élu.
     2. **Réplication des Logs** : Une fois élu, le leader synchronise les commandes avec tous les followers.
     3. **Validation des Logs** : Les entrées dans le log sont confirmées lorsque la majorité des nœuds ont écrit l’entrée dans leur log.
   - **Exemple simplifié en Go** :
     ```go
     type Node struct {
         id      int
         term    int
         isLeader bool
     }

     func (n *Node) requestVote(term int) bool {
         // Simule la requête de vote pour une élection
         if term > n.term {
             n.term = term
             n.isLeader = false
             return true
         }
         return false
     }

     func main() {
         nodes := []Node{{id: 1}, {id: 2}, {id: 3}}
         // Exemple d'initiation d'une élection
         term := 1
         votes := 0
         for _, node := range nodes {
             if node.requestVote(term) {
                 votes++
             }
         }
         if votes > len(nodes)/2 {
             fmt.Println("Leader élu pour le terme", term)
         }
     }
     ```

---

### 3. **Tables de Hachage Distribuées (DHT)**
   - **But des DHT** : Une DHT est utilisée pour distribuer et localiser des données entre plusieurs nœuds sans avoir de serveur central.
   - **Fonctionnement d'une DHT** :
     - Chaque nœud et chaque donnée sont associés à une clé unique, souvent obtenue avec une fonction de hachage.
     - La donnée est stockée sur le nœud ayant une clé proche de celle de la donnée, en fonction de l'algorithme de la DHT.
   - **Exemple d'utilisation** : Stockage de fichiers, recherche de pair-à-pair (comme dans les réseaux BitTorrent).
   - **Implémentation d'une DHT** :
     - **Fonction de Hachage** : On utilise une fonction de hachage pour mapper chaque donnée et nœud sur une plage de valeurs (par exemple, de 0 à 255).
     - **Distribution des données** : Chaque nœud gère une portion de cette plage, et les données sont stockées en fonction de leur clé de hachage.
   - **Exemple simplifié en Go** :
     ```go
     package main

     import (
         "fmt"
         "hash/fnv"
     )

     // Fonction de hachage pour obtenir un identifiant de nœud ou de donnée
     func hashKey(key string) int {
         h := fnv.New32a()
         h.Write([]byte(key))
         return int(h.Sum32() % 256) // Ex: Plage de 0 à 255
     }

     // Simule une DHT avec stockage de données sur un ensemble de nœuds
     type Node struct {
         id    int
         store map[string]string
     }

     func main() {
         nodes := []Node{{id: 100, store: make(map[string]string)},
                         {id: 150, store: make(map[string]string)},
                         {id: 200, store: make(map[string]string)}}

         // Ajouter une donnée et la stocker sur un nœud
         key, value := "filename1", "data"
         hash := hashKey(key)
         for _, node := range nodes {
             if node.id >= hash {
                 node.store[key] = value
                 fmt.Println("Donnée stockée sur le nœud", node.id)
                 break
             }
         }
     }
     ```

---
