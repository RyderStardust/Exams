## Introduction à Go

### 1. Syntaxe de base et déclaration des variables
Go a une syntaxe très épurée. Voici les points essentiels à savoir :
   - **Déclaration de variables** : On utilise `var` pour déclarer des variables avec leur type, ou `:=` pour déclarer et affecter une valeur en même temps (le type est inféré).
     ```go
     var x int = 10 // Déclaration explicite du type
     y := 20        // Déclaration avec inférence de type (Go comprend que y est un entier)
     ```
   - **Types de données** : Go est un langage typé statiquement avec des types basiques comme :
     - `int` pour les entiers
     - `float64` pour les nombres décimaux
     - `string` pour les chaînes de caractères
     - `bool` pour les valeurs vraies ou fausses
     - Des types de collections comme les **tableaux** (`[5]int` pour un tableau de 5 entiers) et les **slices** (`[]int` pour un tableau dynamique).

### 2. Structure de base d’un programme Go
Chaque programme Go commence avec le package `main` et contient une fonction `main()` :
   ```go
   package main

   import "fmt"

   func main() {
       fmt.Println("Hello, Go!")
   }
   ```
   - Le `package main` indique que ce fichier fait partie du programme principal.
   - `import "fmt"` permet d’utiliser le package `fmt` pour afficher des sorties.
   - `fmt.Println("...")` affiche du texte dans la console.

---

## Programmation Réseau en Go

Les communications réseau en Go utilisent principalement le package `net`, qui fournit des outils pour établir des connexions via des protocoles comme TCP et UDP.

### 1. Connexion TCP
Pour établir une connexion TCP, on a besoin de deux composants : un **serveur** qui écoute sur un port et un **client** qui se connecte à ce port.

#### Serveur TCP
   - Le serveur utilise `net.Listen` pour écouter sur un port donné.
   - Lorsqu’il accepte une connexion avec `Accept`, il peut communiquer avec le client.
   - Exécution de `handleConnection` dans une goroutine pour gérer plusieurs connexions simultanément.
   ```go
   package main

   import (
       "fmt"
       "net"
   )

   func main() {
       listener, _ := net.Listen("tcp", ":8080") // Écoute sur le port 8080
       defer listener.Close()                    // Ferme la connexion quand le serveur termine
       for {
           conn, _ := listener.Accept()          // Accepte une connexion
           go handleConnection(conn)             // Lance le gestionnaire dans une goroutine
       }
   }

   func handleConnection(conn net.Conn) {
       fmt.Fprintln(conn, "Hello from server")   // Envoie un message au client
       conn.Close()                              // Ferme la connexion
   }
   ```

#### Client TCP
Le client utilise `net.Dial` pour se connecter au serveur sur l’adresse spécifiée.
   ```go
   package main

   import (
       "fmt"
       "net"
   )

   func main() {
       conn, _ := net.Dial("tcp", "localhost:8080") // Connexion au serveur
       defer conn.Close()                           // Ferme la connexion
       buffer := make([]byte, 1024)
       conn.Read(buffer)                            // Lit la réponse du serveur
       fmt.Println(string(buffer))                  // Affiche la réponse
   }
   ```

---

## Concurrence avec les Goroutines et les Channels

### 1. Les Goroutines
Les **goroutines** permettent de lancer des fonctions en parallèle, de manière très légère.
   ```go
   func sayHello() {
       fmt.Println("Hello")
   }

   func main() {
       go sayHello() // Lance sayHello en tant que goroutine
       fmt.Println("World")
   }
   ```
   Dans cet exemple, `sayHello` est lancé en parallèle avec la goroutine `main`. Cela permet de gérer de nombreuses tâches simultanément sans consommer beaucoup de mémoire.

### 2. Les Channels
Les **channels** permettent de communiquer entre les goroutines. Ils sont utilisés pour envoyer et recevoir des données en synchronisant l'exécution.
   ```go
   func main() {
       ch := make(chan int)      // Crée un channel pour transmettre des entiers
       go func() {
           ch <- 10              // Envoie 10 dans le channel
       }()
       fmt.Println(<-ch)         // Reçoit la valeur depuis le channel
   }
   ```
   Dans cet exemple, le channel `ch` transmet une valeur de la goroutine anonyme vers `main`.

---

## Algorithme de Consensus : Raft

Raft est un algorithme de consensus qui permet aux systèmes distribués de parvenir à un accord sur un ensemble de données. Dans Raft, les nœuds sont organisés en **Leader**, **Followers** et **Candidates**.

### 1. Les Rôles
   - **Leader** : Coordonne le cluster et envoie des commandes aux autres nœuds.
   - **Followers** : Suivent le leader et acceptent ses ordres.
   - **Candidate** : Devient candidate lors d’une élection pour devenir leader.

### 2. Fonctionnement de Raft
   - **Élections** :
     - Quand un follower ne reçoit pas de message du leader, il devient candidate et demande des votes pour devenir leader.
     - Un leader est élu s’il obtient la majorité des votes.
   - **Réplication de Logs** : Une fois élu, le leader envoie des logs (journaux) de commande à chaque follower pour maintenir la cohérence.
   - **Confirmations** : Les commandes sont confirmées lorsque la majorité des nœuds ont écrit cette commande dans leur log.

### Exemple Simplifié en Go
   ```go
   type Node struct {
       id      int
       term    int
       isLeader bool
   }

   func (n *Node) requestVote(term int) bool {
       // Simule une requête de vote pour une élection
       if term > n.term {
           n.term = term
           n.isLeader = false
           return true
       }
       return false
   }

   func main() {
       nodes := []Node{{id: 1}, {id: 2}, {id: 3}}
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

## Tables de Hachage Distribuées (DHT)

Les DHT permettent de stocker et de rechercher des données de manière distribuée en répartissant les données entre plusieurs nœuds.

### 1. Fonctionnement d’une DHT
Chaque nœud a un identifiant unique, souvent basé sur une fonction de hachage. Les données sont également associées à des identifiants, et stockées sur le nœud le plus proche de cet identifiant.

### 2. Utilisation de la fonction de Hachage
   ```go
   package main

   import (
       "fmt"
       "hash/fnv"
   )

   func hashKey(key string) int {
       h := fnv.New32a()
       h.Write([]byte(key))
       return int(h.Sum32() % 256) // Ex : plage de 0 à 255
   }

   type Node struct {
       id    int
       store map[string]string
   }

   func main() {
       nodes := []Node{{id: 100, store: make(map[string]string)},
                       {id: 150, store: make(map[string]string)},
                       {id: 200, store: make(map[string]string)}}

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
   Ici, on utilise une fonction de hachage pour mapper les données sur un nœud en fonction de leur clé.

---
