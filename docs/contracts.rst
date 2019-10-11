.. index:: ! contract

.. _contracts:

##########
Contracts
##########

Les contrats en Solidity sont similaires à des classes dans les langages orientés objets. Ils contiennent des données persistentes dans des variables et des fonctions peuvent les modifier. Appeler la fonction d'un autre contrat (une autre instance) executera un appel de fonction auprès de l'EVM et changera alors le contexte, rendant inaccessibles ces variables.

.. index:: ! contract;creation, constructor

******************
Créer des contrats
******************

Les contrats peuvent être créés "en dehors" via des transactions Ethereum ou depuis des contrat en Solidity.

Les EDIs, comme `Remix <https://remix.ethereum.org/>`_, facilite la tâche via des éléments visuels.

Créer des contrats via du code se fait le plus simplement en utilisant l'API Javascript `web3.js <https://github.com/ethereum/web3.js>`_.
Elle possède une fonction appelée `web3.eth.Contract <https://web3js.readthedocs.io/en/1.0/web3-eth-contract.html#new-contract>`_ qui facilite cette création

Quand un contrat est créé, son constructeur (une fonction déclarée via le mot-clé ``constructor``) est executé, de manière unique.

Un constructeur est optionnel. Aussi, un seul constructeur est autorisé, ce qui signifie que l'overloading n'est pas supporté.

Après que le constructeur ai été exécuté, le code final du contrat est déployé sur la Blockchain. Ce code inclut toutes les fonctions publiques et externes, et toutes les fonctions qui sont atteignables par des appels de fonctions. Le code déployé n'inclut pas le constructeur ou les fonctions internes uniquement appelées depuis le constructeur.

.. index:: constructor;arguments

En interne, les arguments du constructeur sont passés :ref:`ABI encodés <ABI>>` après le code du contrat lui-même, mais vous n'avez pas à vous en soucier si vous utilisez ``web3.js``.

Si un contrat veut créer un autre contrat, le code source (et le binaire) du contrat créé doit être connu du créateur.
Cela signifie que les dépendances cycliques de création sont impossibles.

::

    pragma solidity >=0.4.22 <0.6.0;

    contract OwnedToken {
        // TokenCreator est un type de contrat défini ci-dessous.
        // Il est possible de le référencer tant qu'il n'est pas utilisé
        // pour créer un nouveau contrat.
        TokenCreator creator;
        address owner;
        bytes32 name;

        // Ceci est le constructeur qui enregistre le
        // le créateur et le nom attribué.
        constructor(bytes32 _name) public {
            // Les variables d'état sont accessibles par leur nom
            // et non par l'intermédiaire de this.owner par exemple. Ceci s'applique également
            // aux fonctions et en particulier dans les constructeurs,
            // vous ne pouvez les appeler que comme ça ("en interne"),
            // parce que le contrat lui-même n'existe pas encore.
            owner = msg.sender;
            // Nous effectuons une conversion de type explicite de `address`.
            // vers `TokenCreator` et supposons que le type du
            // contrat appelant est TokenCreator,
            // Il n'y a pas vraiment moyen de vérifier ça.
            creator = TokenCreator(msg.sender);
            name = _name;
        }

        function changeName(bytes32 newName) public {
            // Seul le créateur peut modifier le nom --
            // la comparaison est possible puisque les contrats
            // sont explicitement convertibles en adresses.
            if (msg.sender == address(creator))
                name = newName;
        }

        function transfer(address newOwner) public {
            // Seul le propriétaire actuel peut transférer le token.
            if (msg.sender != owner) return;

            // Nous voulons aussi demander au créateur si le transfert
            // est valide. Notez que ceci appelle une fonction de la fonction
            // contrat défini ci-dessous. Si l'appel échoue (p. ex.
            // en raison d'un manque de gas), l'exécution échoue également ici.
            if (creator.isTokenTransferOK(owner, newOwner))
                owner = newOwner;
        }
    }

    contract TokenCreator {
        function createToken(bytes32 name)
           public
           returns (OwnedToken tokenAddress)
        {
            // Créer un nouveau contrat Token et renvoyer son adresse.
            // Du côté JavaScript, le type de retour est simplement
            // `address`, car c'est le type le plus proche disponible dans
            // l'ABI.
            return new OwnedToken(name);
        }

        function changeName(OwnedToken tokenAddress, bytes32 name) public {
            // Encore une fois, le type externe de `tokenAddress' est
            // simplement `adresse`.
            tokenAddress.changeName(name);
        }

        function isTokenTransferOK(address currentOwner, address newOwner)
            public
            pure
            returns (bool ok)
        {
            // Vérifier une condition arbitraire.
            return keccak256(abi.encodePacked(currentOwner, newOwner))[0] == 0x7f;
        }
    }

.. index:: ! visibility, external, public, private, internal

.. _visibility-and-getters:

**********************
Visibilité et Getters
**********************


Puisque Solidity connaît deux types d'appels de fonction (internes qui ne créent pas d'appel EVM réel (également appelés
a "message call") et externes qui le font), il existe quatre types de visibilités pour les fonctions et les variables d'état.

Les fonctions doivent être spécifiées comme étant ``external``, ``public``, ``internal`` ou ``private``.
Pour les variables d'état, ``external`` n'est pas possible.

``external``:
    Les fonctions externes font partie de l'interface du contrat, ce qui signifie qu'elles peuvent être appelées à partir d'autres contrats et via des transactions. Une fonction externe ``f`` ne peut pas être appelée en interne (c'est-à-dire ``f()``ne fonctionne pas, mais ``this.f()`` fonctionne).
    Les fonctions externes sont parfois plus efficaces lorsqu'elles reçoivent de grandes quantités de données.

``public``:
    Les fonctions publiques font partie de l'interface du contrat et peuvent être appelées en interne ou via des messages. Pour les variables d'état publiques, une fonction getter automatique (voir ci-dessous) est générée.

``internal``:
    Ces fonctions et variables d'état ne sont accessibles qu'en interne (c'est-à-dire à partir du contrat en cours ou des contrats qui en découlent), sans utiliser ``this``.

``private``:
    Les fonctions privées et les variables d'état ne sont visibles que pour le contrat dans lequel elles sont définies et non dans les contrats dérivés.

.. note::
     Tout ce qui se trouve à l'intérieur d'un contrat est visible pour tous les observateurs extérieurs à la blockchain. Passer quelque chose en ``private``
    ne fait qu'empêcher les autres contrats d'accéder à l'information et de la modifier, mais elle sera toujours visible pour le monde entier à l'extérieur de la blockchain.

Le spécificateur de visibilité est donné après le type pour les variables d'état et entre la liste des paramètres et la liste des paramètres de retour pour les fonctions.

::

    pragma solidity >=0.4.16 <0.6.0;

    contract C {
        function f(uint a) private pure returns (uint b) { return a + 1; }
        function setData(uint a) internal { data = a; }
        uint public data;
    }

Dans l'exemple suivant, ``D``, peut appeler ``c.getData()`` pour retrouver la valeur de ``data`` en mémoire d'état, mais ne peut pas appeler ``f``. Le contrat ``E`` est dérivé du contrat ``C`` et peut donc appeler ``compute``.

::

    pragma solidity >=0.4.0 <0.6.0;

    contract C {
        uint private data;

        function f(uint a) private pure returns(uint b) { return a + 1; }
        function setData(uint a) public { data = a; }
        function getData() public view returns(uint) { return data; }
        function compute(uint a, uint b) internal pure returns (uint) { return a + b; }
    }

    // Ceci ne compile pas
    contract D {
        function readData() public {
            C c = new C();
            uint local = c.f(7); // Erreur: le membre `f` n'est pas visible
            c.setData(3);
            local = c.getData();
            local = c.compute(3, 5); // Erreur: le membre `compute` n'est pas visible
        }
    }

    contract E is C {
        function g() public {
            C c = new C();
            uint val = compute(3, 5); // accès à un membre interne (du contrat dérivé au contrat parent)
        }
    }

.. index:: ! getter;function, ! function;getter
.. _getter-functions:

Fonctions Getter
================

Le compilateur crée automatiquement des fonctions getter pour toutes les variables d'état **public**. Pour le contrat donné ci-dessous, le compilateur va générer une fonction appelée ``data`` qui ne prend aucun argument et retourne un ``uint``, la valeur de la variable d'état ``data``. Les variables d'état peuvent être initialisées lorsqu'elles sont déclarées.

::

    pragma solidity >=0.4.0 <0.6.0;

    contract C {
        uint public data = 42;
    }

    contract Caller {
        C c = new C();
        function f() public view returns (uint) {
            return c.data();
        }
    }

Les fonctions getter ont une visibilité externe. Si le symbole est accédé en interne (c'est-à-dire sans ``this.``), il est évalué à une variable d'état.  S'il est accédé de l'extérieur (c'est-à-dire avec ``this.``), il évalue à une fonction.

::

    pragma solidity >=0.4.0 <0.6.0;

    contract C {
        uint public data;
        function x() public returns (uint) {
            data = 3; // accès interne
            return this.data(); // accès externe
        }
    }

Si vous avez une variable d'état ``public`` de type array, alors vous ne pouvez récupérer que des éléments simples de l'array via la fonction getter générée. Ce mécanisme permet d'éviter des coûts de gas élevés lors du retour d'un tableau complet. Vous pouvez utiliser des arguments pour spécifier quel élément individuel retourner, par exemple ``data(0)``. Si vous voulez retourner un tableau entier en un appel, alors vous devez écrire une fonction, par exemple :

::

  pragma solidity >=0.4.0 <0.6.0;

  contract arrayExample {
    // variable d'état publique
    uint[] public myArray;

    // Fonction getter générée par le compilateur
    /*
    function myArray(uint i) returns (uint) {
        return myArray[i];
    }
    */

    // fonction retournant une array complète
    function getArray() returns (uint[] memory) {
        return myArray;
    }
  }

Maintenant vous pouvez utiliser ``getArray()`` pour récupérer le tableau entier, au lieu de ``myArray(i)``, qui retourne un seul élément par appel.

L'exemple suivant est plus complexe:

::

    pragma solidity >=0.4.0 <0.6.0;

    contract Complex {
        struct Data {
            uint a;
            bytes3 b;
            mapping (uint => uint) map;
        }
        mapping (uint => mapping(bool => Data[])) public data;
    }

Il génère une fonction de la forme suivante. Le mappage dans la structure est omis parce qu'il n'y a pas de bonne façon de fournir la clé pour le mappage :

::

    function data(uint arg1, bool arg2, uint arg3) public returns (uint a, bytes3 b) {
        a = data[arg1][arg2][arg3].a;
        b = data[arg1][arg2][arg3].b;
    }

.. index:: ! function;modifier

.. _modifiers:

******************
Modificateurs de fonctions
******************

Les modificateurs peuvent être utilisés pour modifier facilement le comportement des fonctions.  Par exemple, ils peuvent vérifier automatiquement une condition avant d'exécuter la fonction. Les modificateurs sont des propriétés héritables des contrats et peuvent être redéfinis dans les contrats dérivés.

::

    pragma solidity >0.4.99 <0.6.0;

    contract owned {
        constructor() public { owner = msg.sender; }
        address payable owner;

        // Ce contrat ne définit qu'un modificateur mais ne l'utilise pas:
        // il sera utilisé dans les contrats dérivés.
        // Le corps de la fonction est inséré à l'endroit où le symbole spécial
        // `_;` apparaît dans la définition d'un modificateur.
        // Cela signifie que si le propriétaire appelle cette fonction, la fonction
        // est exécutée et dans le cas contraire, une exception est
        // levée.
        modifier onlyOwner {
            require(
                msg.sender == owner,
                "Only owner can call this function."
            );
            _;
        }
    }

    contract mortal is owned {
        // Ce contrat hérite du modificateur `onlyOwner` de `owned`
        // et l'applique à la fonction `close`, qui
        // cause que les appels à `close` n'ont un effet que s'il
        // sont passés par le propriétaire enregistré.
        function close() public onlyOwner {
            selfdestruct(owner);
        }
    }

    contract priced {
        // Les modificateurs peuvent prendre des arguments:
        modifier costs(uint price) {
            if (msg.value >= price) {
                _;
            }
        }
    }

    contract Register is priced, owned {
        mapping (address => bool) registeredAddresses;
        uint price;

        constructor(uint initialPrice) public { price = initialPrice; }

        // Il est important de fournir également le
        // mot-clé `payable` ici, sinon la fonction
        // rejettera automatiquement tous les Ethers qui lui sont envoyés.
        function register() public payable costs(price) {
            registeredAddresses[msg.sender] = true;
        }

        function changePrice(uint _price) public onlyOwner {
            price = _price;
        }
    }

    contract Mutex {
        bool locked;
        modifier noReentrancy() {
            require(
                !locked,
                "Reentrant call."
            );
            locked = true;
            _;
            locked = false;
        }

        /// Cette fonction est protégée par un mutex, ce qui signifie que
        /// les appels entrants à partir de `msg.sender.call` ne peuvent pas rappeler `f`.
        /// L'instruction `return 7` assigne 7 à la valeur de retour, mais en même temps
        /// exécute l'instruction `locked = false` dans le modificateur.
        function f() public noReentrancy returns (uint) {
            (bool success,) = msg.sender.call("");
            require(success);
            return 7;
        }
    }

Plusieurs modificateurs sont appliqués à une fonction en les spécifiant dans une liste séparée par des espaces et sont évalués dans l'ordre présenté.

.. avertissement::
    Dans une version antérieure de Solidity, les instructions ``return`` des fonctions ayant des modificateurs se comportaient différemment.

Les retours explicites d'un modificateur ou d'un corps de fonction ne laissent que le modificateur ou le corps de fonction courant. Les variables de retour sont affectées et le flow de contrôle continue après le "_" dans le modificateur précédent.

Des expressions arbitraires sont autorisées pour les arguments du modificateur et dans ce contexte, tous les symboles visibles depuis la fonction sont visibles dans le modificateur. Les symboles introduits dans le modificateur ne sont pas visibles dans la fonction (car ils peuvent changer en cas de redéfinition).

Traduit avec www.DeepL.com/Translator

.. index:: ! constant

************************
Variables d'état constantes
************************

Les variables d'état peuvent être déclarées comme ``constantes``. Dans ce cas, elles doivent être assignées à partir d'une expression constante au moment de la compilation. Toute expression qui accède au stockage, aux données de la blockchain (par exemple ``now``, ``address(this).balance`` ou ``block.number``) ou
les données d'exécution (``msg.value`` ou ``gasleft()``) ou les appels vers des contrats externes sont interdits. Les expressions qui peuvent avoir un effet secondaire sur l'allocation de mémoire sont autorisées, mais celles qui peuvent avoir un effet secondaire sur d'autres objets mémoire ne le sont pas. Les fonctions intégrées ``keccak256``, ``sha256``, ``ripemd160``, ``ecrecover``, ``addmod`` et ``mulmod`` sont autorisées (même si des contrats externes sont appelés).

La raison pour laquelle on autorise les effets secondaires sur l'allocateur de mémoire est qu'il devrait être possible de construire des objets complexes comme par exemple des tables de consultation.
Cette fonctionnalité n'est pas encore entièrement utilisable.

Le compilateur ne réserve pas d'emplacement de stockage pour ces variables, et chaque occurrence est remplacée par l'expression constante correspondante (qui peut être calculée à une valeur unique par l'optimiseur).

Tous les types de constantes ne sont pas implémentés pour le moment. Les seuls types pris en charge sont les types valeurs et les chaînes de caractères.

::

    pragma solidity >=0.4.0 <0.6.0;

    contract C {
        uint constant x = 32**22 + 8;
        string constant text = "abc";
        bytes32 constant myHash = keccak256("abc");
    }

.. index:: ! functions

.. _functions:

*********
Fonctions
*********

.. index:: ! view function, function;view

.. _view-functions:

Fonctions View
==============

Les fonctions peuvent être déclarées ``view``, auquel cas elles promettent de ne pas modifier l'état.

.. note::
  Si la cible EVM du compilateur est Byzantium ou plus récent (par défaut), l'opcode ``STATICCALL`` est utilisé pour les fonctions ``view`` qui imposent à l'état de rester non modifié lors de l'exécution EVM. Pour les librairies, on utilise les fonctions ``view`` et ``DELEGATECALL`` parce qu'il n'y a pas de ``DELEGATECALL`` et ``STATICCALL`` combinés.
  Cela signifie que les fonctions ``view`` de librairies n'ont pas de contrôles d'exécution qui empêchent les modifications d'état. Cela ne devrait pas avoir d'impact négatif sur la sécurité car le code de librairies est généralement connu au moment de la compilation et le vérificateur statique effectue les vérifications au moment de la compilation.

Les déclarations suivantes sont considérées comme une modification de l'état :

#. Ecrire dans les variables d'état.
#. :ref:`Emettre des événements <events>`.
#. :ref:`Création d'autres contrats <creating-contracts>`.
#. Utiliser ``selfdestruct``.
#. Envoyer des Ethers par des appels.
#. Appeler une fonction qui n'est pas marquée ``view`` ou ``pure``.
#. Utilisation d'appels bas niveau.
#. Utilisation d'assembleur inline qui contient certains opcodes.

::

    pragma solidity >0.4.99 <0.6.0;

    contract C {
        function f(uint a, uint b) public view returns (uint) {
            return a * (b + 42) + now;
        }
    }

.. note::
  ``constant`` sur les fonctions était un alias de ``view``, mais cela a été abandonné dans la version 0.5.0.

.. note::
  Les méthodes Getter sont automatiquement marquées ``view``.

.. note::
  Avant la version 0.5.0, le compilateur n'utilisait pas l'opcode ``STATICCALL``.
  pour les fonctions ``view``.
  Cela permettait de modifier l'état des fonctions ``view`` grâce à l'utilisation de
  conversions de type explicites non valides.
  En utilisant ``STATICCALL`` pour les fonctions ``view``, les modifications de la fonction
  sont évités au niveau de l'EVM.

.. index:: ! pure function, function;pure

.. _pure-functions:

Fonctions Pure
==============

Les fonctions peuvent être déclarées ``pures``, auquel cas elles promettent de ne pas lire ou modifier l'état.

.. note::
  Si la cible EVM du compilateur est Byzantium ou plus récente (par défaut), on utilise l'opcode ``STATICCALL``, ce qui ne garantit pas que l'état ne soit pas lu, mais au moins qu'il ne soit pas modifié.

En plus de la liste des modificateurs d'état expliqués ci-dessus, sont considérés comme des lectures de l'état :

#. Lecture des variables d'état.
#. Accéder à ``address(this).balance`` ou ``<address>.balance``.
#. Accéder à l'un des membres de ``block``, ``tx``, ``msg`` (à l'exception de ``msg.sig`` et ``msg.data``).
#. Appeler une fonction qui n'est pas marquée ``pure``.
#. Utilisation d'assembleur inline qui contient certains opcodes.

::

    pragma solidity >0.4.99 <0.6.0;

    contract C {
        function f(uint a, uint b) public pure returns (uint) {
            return a * (b + 42);
        }
    }

.. note::
  Avant la version 0.5.0, le compilateur n'utilisait pas l'opcode ``STATICCALL`` pour les fonctions ``pure``.
  Cela permettait de modifier l'état des fonctions ``pures`` en utilisant des conversions de type explicites invalides.
  En utilisant ``STATICCALL`` pour des fonctions ``pures``, les modifications de l'état sont empêchées au niveau de l'EVM.

.. avertissement::
  Il n'est pas possible d'empêcher les fonctions de lire l'état au niveau de l'EVM, il est seulement possible de les empêcher d'écrire dans l'état (c'est-à-dire que seul "view" peut être exécuté au niveau de l'EVM, ``pure`` ne peut pas).

.. avertissement::
  Avant la version 0.4.17, le compilateur n'appliquait pas le fait que ``pure`` ne lisait pas l'état.
  Il s'agit d'un contrôle de type à la compilation, qui peut être contourné en effectuant des conversions explicites invalides entre les types de contrats, parce que le compilateur peut vérifier que le type de contrat ne fait pas d'opérations de changement d'état, mais il ne peut pas vérifier que le contrat qui sera appelé à l'exécution est effectivement de ce type.

.. index:: ! fallback function, function;fallback

.. _fallback-function:

Fonction de repli
=================

Un contrat peut avoir exactement une fonction sans nom. Cette fonction ne peut pas avoir d'arguments, ne peut rien retourner et doit avoir une visibilité ``external``.
Elle est exécutée lors d'un appel au contrat si aucune des autres fonctions ne correspond à l'identificateur de fonction donné (ou si aucune donnée n'a été fournie).

En outre, cette fonction est exécutée chaque fois que le contrat reçoit des Ethers bruts (sans données). De plus, pour recevoir des Ethers, la fonction de fallback doit être marquée ``payable``. En l'absence d'une telle fonction, le contrat ne peut recevoir
d'Ether par des transactions traditionnelles.

Dans le pire des cas, la fonction de fallback ne peut compter que sur la disponibilité de 2 300 gas (par exemple lorsque l'on utilise `send` ou `transfer`), ce qui laisse peu de place pour effectuer d'autres opérations que du log basique. Les opérations suivantes
consommeront plus de gaz que le forfait de 2 300 gas alloué :

- Ecrire dans le stockage
- Création d'un contrat
- Appel d'une fonction externe qui consomme une grande quantité de gas
- Envoi d'Ether

Comme toute fonction, la fonction de fallback peut exécuter des opérations complexes tant que suffisamment de gas lui est transmis.

.. note::
    Même si la fonction de fallback ne peut pas avoir d'arguments, on peut toujours utiliser ``msg.data`` pour récupérer toute charge utile fournie avec l'appel.

.. avertissement::
    La fonction de fallback est également exécutée si l'appelant a l'intention d'appeler une fonction qui n'est pas disponible. Si vous voulez implémenter la fonction de fallback uniquement pour recevoir de l'Ether, vous devez ajouter une vérification comme ``require(msg.data.length == 0)`` pour éviter les appels invalides.

.. avertissement::
    Les contrats qui reçoivent directement l'Ether (sans appel de fonction, c'est-à-dire en utilisant ``send`` ou `` transfer``) mais ne définissent pas de fonction de fallback lèvent une exception, renvoyant l'Ether (c'était différent avant Solidity v0.4.0). Donc si vous voulez que votre contrat reçoive de l'Ether, vous devez implémenter une fonction de fallback ``payable``.

.. avertissement::
    Un contrat sans fonction de fallback payable peut recevoir de l'Ether en tant que destinataire d'une `coinbase transaction` (alias `récompense de mineur de bloc`) ou en tant que destination d'un ``selfdestruct``.

    Un contrat ne peut pas réagir à de tels transferts d'Ether et ne peut donc pas non plus les rejeter. C'est un choix de conception de l'EVM et Solidity ne peut le contourner.

    Cela signifie également que ``address(this).balance`` peut être plus élevé que la somme de certaines comptabilités manuelles implémentées dans un contrat (i.e. avoir un compteur mis à jour dans la fonction fallback).

::

    pragma solidity >0.4.99 <0.6.0;

    contract Test {
        // This function is called for all messages sent to
        // this contract (there is no other function).
        // Sending Ether to this contract will cause an exception,
        // because the fallback function does not have the `payable`
        // modifier.
        function() external { x = 1; }
        uint x;
    }


    // This contract keeps all Ether sent to it with no way
    // to get it back.
    contract Sink {
        function() external payable { }
    }

    contract Caller {
        function callTest(Test test) public returns (bool) {
            (bool success,) = address(test).call(abi.encodeWithSignature("nonExistingFunction()"));
            require(success);
            // results in test.x becoming == 1.

            // address(test) will not allow to call ``send`` directly, since ``test`` has no payable
            // fallback function. It has to be converted to the ``address payable`` type via an
            // intermediate conversion to ``uint160`` to even allow calling ``send`` on it.
            address payable testPayable = address(uint160(address(test)));

            // If someone sends ether to that contract,
            // the transfer will fail, i.e. this returns false here.
            return testPayable.send(2 ether);
        }
    }

.. index:: ! overload

.. _overload-function:

Surcharge de  (overload)
====================

Un contrat peut avoir plusieurs fonctions du même nom, mais avec des types de paramètres différents.
Ce processus est appelé "surcharge" et s'applique également aux fonctions héritées.
L'exemple suivant montre la surcharge de la fonction ``f`` dans le champ d'application du contrat ``A``.

::

    pragma solidity >=0.4.16 <0.6.0;

    contract A {
        function f(uint _in) public pure returns (uint out) {
            out = _in;
        }

        function f(uint _in, bool _really) public pure returns (int out) {
            if (_really)
                out = int(_in);
        }
    }

Des fonctions surchargées sont également présentes dans l'interface externe. C'est une erreur si deux fonctions visibles de l'extérieur diffèrent par leur type Solidity (ici `A` et `B`) mais pas par leur type extérieur (ici `address``).

::

    pragma solidity >=0.4.16 <0.6.0;

    // Ceci ne compile pas
    contract A {
        function f(B _in) public pure returns (B out) {
            out = _in;
        }

        function f(A _in) public pure returns (B out) {
            out = B(address(_in));
        }
    }

    contract B {
    }


Les deux fonctions ``f`` surchargées ci-dessus acceptent des addresses du point de vue de l'ABI, mais ces adresses sont considérées comme différents types en Solidity.

Résolution des surcharges et concordance des arguments
-----------------------------------------

Les fonctions surchargées sont sélectionnées en faisant correspondre les déclarations de fonction dans le scope actuel aux arguments fournis dans l'appel de fonction. La fonction évaluée est choisie si tous les arguments peuvent être implicitement convertis en types attendus. S'il y a plusieurs fonctions correspondantes, la résolution échoue.

.. note::
    Le type des valeurs retournées par la fonction n'est pas pris en compte dans la résolution des surcharges.

::

    pragma solidity >=0.4.16 <0.6.0;

    contract A {
        function f(uint8 _in) public pure returns (uint8 out) {
            out = _in;
        }

        function f(uint256 _in) public pure returns (uint256 out) {
            out = _in;
        }
    }

L'appel de ``f(50)`` créerait une erreur de type puisque ``50`` peut être implicitement converti à la fois en type ``uint8`` et ``uint256``. D'un autre côté, ``f(256)`` se résoudrait à ``f(uint256)`` car ``256`` ne peut pas être implicitement converti en ``uint8``.

.. index:: ! event

.. _events:

******
Événements
******

Les événements Solidity autorisent une abstraction en plus de la fonctionnalité de journalisation de l'EVM.
Les applications peuvent souscrire à et écouter ces événements via l'interface RPC d'un client Ethereum.

Les événements sont des membres héritables des contrats. Lorsque vous les appelez, ils font en sorte que les arguments soient stockés dans le journal des transactions - une structure de données spéciale dans la blockchain. Ces logs sont associés à l'adresse du contrat, sont incorporés dans la blockchain et y restent tant qu'un bloc est accessible (pour toujours à partir des versions Frontier et Homestead, mais cela peut changer avec Serenity). Le journal et ses données d'événement ne sont pas accessibles depuis les contrats (pas même depuis le contrat qui les a créés).

Il est possible de demander une simple vérification de paiement (SPV) pour les logs, de sorte que si une entité externe fournit un contrat avec une telle vérification, elle peut vérifier que le log existe réellement dans la blockchain. Vous devez fournir des en-têtes (headers) de bloc car le contrat ne peut voir que les 256 derniers hashs de blocs.

Vous pouvez ajouter l'attribut ``indexed`` à un maximum de trois paramètres qui les ajoute à une structure de données spéciale appelée :ref:`"topics" <abi_events>` au lieu de la partie data du log. Si vous utilisez des tableaux (y compris les ``string`` et ``bytes``)
comme arguments indexés, leurs hashs Keccak-256 sont stockés comme topic à la place, car un topic ne peut contenir qu'un seul mot (32 octets).

Tous les paramètres sans l'attribut ``indexed`` sont :ref:`ABI-encoded <ABI>` dans la partie données du log.

Les topics vous permettent de rechercher des événements, par exemple lors du filtrage d'une séquence de blocs pour certains événements. Vous pouvez également filtrer les événements par l'adresse du contrat qui les a émis.

Par exemple, le code ci-dessous utilise web3.js ``subscribe("logs")``
`method <https://web3js.readthedocs.io/en/1.0/web3-eth-subscribe.html#subscribe-logs>`_  pour filtrer les logs qui correspondent à un sujet avec une certaine valeur d'adresse :

.. code-block:: javascript

    var options = {
        fromBlock: 0,
        address: web3.eth.defaultAccount,
        topics: ["0x0000000000000000000000000000000000000000000000000000000000000000", null, null]
    };
    web3.eth.subscribe('logs', options, function (error, result) {
        if (!error)
            console.log(result);
    })
        .on("data", function (log) {
            console.log(log);
        })
        .on("changed", function (log) {
    });


Le hash de la signature de l'event est l'un des topics, sauf si vous avez déclaré l'événement avec le spécificateur "anonymous". Cela signifie qu'il n'est pas possible de filtrer des événements anonymes spécifiques par leur nom.

::

    pragma solidity >=0.4.21 <0.6.0;

    contract ClientReceipt {
        event Deposit(
            address indexed _from,
            bytes32 indexed _id,
            uint _value
        );

        function deposit(bytes32 _id) public payable {
            // Les événements sont émis à l'aide de `emit`, suivi du
            // nom de l'événement et des arguments
            // (le cas échéant) entre parenthèses. Une telle invocation
            // (même profondément imbriquée) peut être détectée à partir de
            // l'API JavaScript en filtrant `Deposit`.
            emit Deposit(msg.sender, _id, msg.value);
        }
    }

L'utilisation dans l'API JavaScript est la suivante :

::

    var abi = /* abi telle que génerée par le compilateur */;
    var ClientReceipt = web3.eth.contract(abi);
    var clientReceipt = ClientReceipt.at("0x1234...ab67" /* address */);

    var event = clientReceipt.Deposit();

    // watch for changes
    event.watch(function(error, result){
        // result contains non-indexed arguments and topics
        // given to the `Deposit` call.
        if (!error)
            console.log(result);
    });


    // Or pass a callback to start watching immediately
    var event = clientReceipt.Deposit(function(error, result) {
        if (!error)
            console.log(result);
    });

The output of the above looks like the following (trimmed):

.. code-block:: json

  {
     "returnValues": {
         "_from": "0x1111…FFFFCCCC",
         "_id": "0x50…sd5adb20",
         "_value": "0x420042"
     },
     "raw": {
         "data": "0x7f…91385",
         "topics": ["0xfd4…b4ead7", "0x7f…1a91385"]
     }
  }

.. index:: ! log

Low-Level Interface to Logs
===========================

It is also possible to access the low-level interface to the logging mechanism via the functions ``log0``, ``log1``, ``log2``, ``log3`` and ``log4``.
``logi`` takes ``i + 1`` parameter of type ``bytes32``, where the first argument will be used for the data part of the log and the others as topics. The event call above can be performed in the same way as

::

    pragma solidity >=0.4.10 <0.6.0;

    contract C {
        function f() public payable {
            uint256 _id = 0x420042;
            log3(
                bytes32(msg.value),
                bytes32(0x50cb9fe53daa9737b786ab3646f04d0150dc50ef4e75f59509d83667ad5adb20),
                bytes32(uint256(msg.sender)),
                bytes32(_id)
            );
        }
    }

where the long hexadecimal number is equal to ``keccak256("Deposit(address,bytes32,uint256)")``, the signature of the event.

Additional Resources for Understanding Events
==============================================

- `Javascript documentation <https://github.com/ethereum/wiki/wiki/JavaScript-API#contract-events>`_
- `Example usage of events <https://github.com/debris/smart-exchange/blob/master/lib/contracts/SmartExchange.sol>`_
- `How to access them in js <https://github.com/debris/smart-exchange/blob/master/lib/exchange_transactions.js>`_

.. index:: ! inheritance, ! base class, ! contract;base, ! deriving

***********
Inheritance
***********

Solidity supports multiple inheritance by copying code including polymorphism.

All function calls are virtual, which means that the most derived function is called, except when the contract name is explicitly given.

When a contract inherits from other contracts, only a single
contract is created on the blockchain, and the code from all the base contracts
is copied into the created contract.

The general inheritance system is very similar to `Python's <https://docs.python.org/3/tutorial/classes.html#inheritance>`_,
especially concerning multiple inheritance, but there are also some :ref:`differences <multi-inheritance>`.

Details are given in the following example.

::

    pragma solidity >0.4.99 <0.6.0;

    contract owned {
        constructor() public { owner = msg.sender; }
        address payable owner;
    }

    // Use `is` to derive from another contract. Derived contracts can access all non-private members including internal functions and state variables. These cannot be accessed externally via `this`, though.
    contract mortal is owned {
        function kill() public {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    // These abstract contracts are only provided to make the interface known to the compiler. Note the function without body. If a contract does not implement all functions it can only be used as an interface.
    contract Config {
        function lookup(uint id) public returns (address adr);
    }

    contract NameReg {
        function register(bytes32 name) public;
        function unregister() public;
     }

    // Multiple inheritance is possible. Note that `owned` is also a base class of `mortal`, yet there is only a single instance of `owned` (as for virtual inheritance in C++).
    contract named is owned, mortal {
        constructor(bytes32 name) public {
            Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
            NameReg(config.lookup(1)).register(name);
        }

        // Functions can be overridden by another function with the same name and the same number/types of inputs.  If the overriding function has different types of output parameters, that causes an error.
        // Both local and message-based function calls take these overrides into account.
        function kill() public {
            if (msg.sender == owner) {
                Config config = Config(0xD5f9D8D94886E70b06E474c3fB14Fd43E2f23970);
                NameReg(config.lookup(1)).unregister();
                // It is still possible to call a specific overridden function.
                mortal.kill();
            }
        }
    }

    // If a constructor takes an argument, it needs to be provided in the header (or modifier-invocation-style at the constructor of the derived contract (see below)).
    contract PriceFeed is owned, mortal, named("GoldFeed") {
       function updateInfo(uint newInfo) public {
          if (msg.sender == owner) info = newInfo;
       }

       function get() public view returns(uint r) { return info; }

       uint info;
    }

Note that above, we call ``mortal.kill()`` to "forward" the destruction request. The way this is done is problematic, as
seen in the following example::

    pragma solidity >=0.4.22 <0.6.0;

    contract owned {
        constructor() public { owner = msg.sender; }
        address payable owner;
    }

    contract mortal is owned {
        function kill() public {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    contract Base1 is mortal {
        function kill() public { /* do cleanup 1 */ mortal.kill(); }
    }

    contract Base2 is mortal {
        function kill() public { /* do cleanup 2 */ mortal.kill(); }
    }

    contract Final is Base1, Base2 {
    }

A call to ``Final.kill()`` will call ``Base2.kill`` as the most derived override, but this function will bypass ``Base1.kill``, basically because it does not even know about ``Base1``.  The way around this is to use ``super``::

    pragma solidity >=0.4.22 <0.6.0;

    contract owned {
        constructor() public { owner = msg.sender; }
        address payable owner;
    }

    contract mortal is owned {
        function kill() public {
            if (msg.sender == owner) selfdestruct(owner);
        }
    }

    contract Base1 is mortal {
        function kill() public { /* do cleanup 1 */ super.kill(); }
    }


    contract Base2 is mortal {
        function kill() public { /* do cleanup 2 */ super.kill(); }
    }

    contract Final is Base1, Base2 {
    }

If ``Base2`` calls a function of ``super``, it does not simply call this function on one of its base contracts.  Rather, it calls this function on the next base contract in the final inheritance graph, so it will call ``Base1.kill()`` (note that the final inheritance sequence is -- starting with the most derived contract: Final, Base2, Base1, mortal, owned).
The actual function that is called when using super is not known in the context of the class where it is used, although its type is known. This is similar for ordinary virtual method lookup.

.. index:: ! constructor

.. _constructor:

Constructors
============

A constructor is an optional function declared with the ``constructor`` keyword which is executed upon contract creation, and where you can run contract initialisation code.

Before the constructor code is executed, state variables are initialised to their specified value if you initialise them inline, or zero if you do not.

After the constructor has run, the final code of the contract is deployed to the blockchain. The deployment of the code costs additional gas linear to the length of the code.
This code includes all functions that are part of the public interface and all functions that are reachable from there through function calls.
It does not include the constructor code or internal functions that are only called from the constructor.

Constructor functions can be either ``public`` or ``internal``. If there is no constructor, the contract will assume the default constructor, which is equivalent to ``constructor() public {}``. For example:

::

    pragma solidity >0.4.99 <0.6.0;

    contract A {
        uint public a;

        constructor(uint _a) internal {
            a = _a;
        }
    }

    contract B is A(1) {
        constructor() public {}
    }

A constructor set as ``internal`` causes the contract to be marked as :ref:`abstract <abstract-contract>`.

.. warning ::
    Prior to version 0.4.22, constructors were defined as functions with the same name as the contract.
    This syntax was deprecated and is not allowed anymore in version 0.5.0.


.. index:: ! base;constructor

Arguments for Base Constructors
===============================

The constructors of all the base contracts will be called following the linearization rules explained below. If the base constructors have arguments, derived contracts need to specify all of them. This can be done in two ways::

    pragma solidity >=0.4.22 <0.6.0;

    contract Base {
        uint x;
        constructor(uint _x) public { x = _x; }
    }

    // Either directly specify in the inheritance list...
    contract Derived1 is Base(7) {
        constructor() public {}
    }

    // or through a "modifier" of the derived constructor.
    contract Derived2 is Base {
        constructor(uint _y) Base(_y * _y) public {}
    }

One way is directly in the inheritance list (``is Base(7)``).  The other is in the way a modifier is invoked as part of
the derived constructor (``Base(_y * _y)``). The first way to do it is more convenient if the constructor argument is a
constant and defines the behaviour of the contract or describes it. The second way has to be used if the constructor arguments of the base depend on those of the derived contract. Arguments have to be given either in the inheritance list or in modifier-style in the derived constructor.
Specifying arguments in both places is an error.

If a derived contract does not specify the arguments to all of its base contracts' constructors, it will be abstract.

.. index:: ! inheritance;multiple, ! linearization, ! C3 linearization

.. _multi-inheritance:

Multiple Inheritance and Linearization
======================================

Languages that allow multiple inheritance have to deal with several problems.  One is the `Diamond Problem <https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem>`_.
Solidity is similar to Python in that it uses "`C3 Linearization <https://en.wikipedia.org/wiki/C3_linearization>`_" to force a specific order in the directed acyclic graph (DAG) of base classes. This results in the desirable property of monotonicity but disallows some inheritance graphs. Especially, the order in which the base classes are given in the ``is`` directive is
important: You have to list the direct base contracts in the order from "most base-like" to "most derived".
Note that this order is the reverse of the one used in Python.

Another simplifying way to explain this is that when a function is called that is defined multiple times in different contracts, the given bases are searched from right to left (left to right in Python) in a depth-first manner, stopping at the first match. If a base contract has already been searched, it is skipped.

In the following code, Solidity will give theerror "Linearization of inheritance graph impossible".

::

    pragma solidity >=0.4.0 <0.6.0;

    contract X {}
    contract A is X {}
    // This will not compile
    contract C is A, X {}

The reason for this is that ``C`` requests ``X`` to override ``A`` (by specifying ``A, X`` in this order), but ``A`` itself
requests to override ``X``, which is a contradiction that cannot be resolved.



Inheriting Different Kinds of Members of the Same Name
======================================================

When the inheritance results in a contract with a function and a modifier of the same name, it is considered as an error.
This error is produced also by an event and a modifier of the same name, and a function and an event of the same name.
As an exception, a state variable getter can override a public function.

.. index:: ! contract;abstract, ! abstract contract

.. _abstract-contract:

******************
Abstract Contracts
******************

Contracts are marked as abstract when at least one of their functions lacks an implementation as in the following example (note that the function declaration header is terminated by ``;``)::

    pragma solidity >=0.4.0 <0.6.0;

    contract Feline {
        function utterance() public returns (bytes32);
    }

Such contracts cannot be compiled (even if they contain implemented functions alongside non-implemented functions), but they can be used as base contracts::

    pragma solidity >=0.4.0 <0.6.0;

    contract Feline {
        function utterance() public returns (bytes32);
    }

    contract Cat is Feline {
        function utterance() public returns (bytes32) { return "miaow"; }
    }

If a contract inherits from an abstract contract and does not implement all non-implemented functions by overriding, it will itself be abstract.

Note that a function without implementation is different from a :ref:`Function Type <function_types>` even though their syntax looks very similar.

Example of function without implementation (a function declaration)::

    function foo(address) external returns (address);

Example of a Function Type (a variable declaration, where the variable is of type ``function``)::

    function(address) external returns (address) foo;

Abstract contracts decouple the definition of a contract from its implementation providing better extensibility and self-documentation and facilitating patterns like the `Template method <https://en.wikipedia.org/wiki/Template_method_pattern>`_ and removing code duplication.
Abstract contracts are useful in the same way that defining methods in an interface is useful. It is a way for the designer of the abstract contract to say "any child of mine must implement this method".


.. index:: ! contract;interface, ! interface contract

.. _interfaces:

**********
Interfaces
**********

Interfaces are similar to abstract contracts, but they cannot have any functions implemented. There are further restrictions:

- They cannot inherit other contracts or interfaces.
- All declared functions must be external.
- They cannot declare a constructor.
- They cannot declare state variables.

Some of these restrictions might be lifted in the future.

Interfaces are basically limited to what the Contract ABI can represent, and the conversion between the ABI and an interface should be possible without any information loss.

Interfaces are denoted by their own keyword:

::

    pragma solidity >=0.4.11 <0.6.0;

    interface Token {
        enum TokenType { Fungible, NonFungible }
        struct Coin { string obverse; string reverse; }
        function transfer(address recipient, uint amount) external;
    }

Contracts can inherit interfaces as they would inherit other contracts.

Types defined inside interfaces and other contract-like structures can be accessed from other contracts: ``Token.TokenType`` or ``Token.Coin``.

.. index:: ! library, callcode, delegatecall

.. _libraries:

*********
Libraries
*********

Libraries are similar to contracts, but their purpose is that they are deployed only once at a specific address and their code is reused using the ``DELEGATECALL`` (``CALLCODE`` until Homestead) feature of the EVM. This means that if library functions are called, their code is executed in the context of the calling contract, i.e. ``this`` points to the calling contract, and especially the storage from the calling contract can be accessed. As a library is an isolated piece of source code, it can only access state variables of the calling contract if they are explicitly supplied (it would have no way to name them, otherwise). Library functions can only be called directly (i.e. without the use of ``DELEGATECALL``) if they do not modify the state (i.e. if they are ``view`` or ``pure`` functions), because libraries are assumed to be stateless. In particular, it is not possible to destroy a library.

.. note::
    Until version 0.4.20, it was possible to destroy libraries by circumventing Solidity's type system. Starting from that version, libraries contain a :ref:`mechanism<call-protection>` that disallows state-modifying functions to be called directly (i.e. without ``DELEGATECALL``).

Libraries can be seen as implicit base contracts of the contracts that use them.
They will not be explicitly visible in the inheritance hierarchy, but calls to library functions look just like calls to functions of explicit base contracts (``L.f()`` if ``L`` is the name of the library). Furthermore, ``internal`` functions of libraries are visible in all contracts, just as if the library were a base contract. Of course, calls to internal functions
use the internal calling convention, which means that all internal types can be passed and types :ref:`stored in memory <data-location>` will be passed by reference and not copied.
To realize this in the EVM, code of internal library functions and all functions called from therein will at compile time be pulled into the calling contract, and a regular ``JUMP`` call will be used instead of a ``DELEGATECALL``.

.. index:: using for, set

The following example illustrates how to use libraries (but manual method be sure to check out :ref:`using for <using-for>` for a more advanced example to implement a set).

::

    pragma solidity >=0.4.22 <0.6.0;

    library Set {
      // We define a new struct datatype that will be used to hold its data in the calling contract.
      struct Data { mapping(uint => bool) flags; }

      // Note that the first parameter is of type "storage reference" and thus only its storage address and not its contents is passed as part of the call.  This is a special feature of library functions.  It is idiomatic to call the first parameter `self`, if the function can be seen as a method of that object.
      function insert(Data storage self, uint value)
          public
          returns (bool)
      {
          if (self.flags[value])
              return false; // already there
          self.flags[value] = true;
          return true;
      }

      function remove(Data storage self, uint value)
          public
          returns (bool)
      {
          if (!self.flags[value])
              return false; // not there
          self.flags[value] = false;
          return true;
      }

      function contains(Data storage self, uint value)
          public
          view
          returns (bool)
      {
          return self.flags[value];
      }
    }

    contract C {
        Set.Data knownValues;

        function register(uint value) public {
            // The library functions can be called without a specific instance of the library, since the "instance" will be the current contract.
            require(Set.insert(knownValues, value));
        }
        // In this contract, we can also directly access knownValues.flags, if we want.
    }

Of course, you do not have to follow this way to use libraries: they can also be used without defining struct data types. Functions also work without any storage reference parameters, and they can have multiple storage reference parameters and in any position.

The calls to ``Set.contains``, ``Set.insert`` and ``Set.remove`` are all compiled as calls (``DELEGATECALL``) to an external
contract/library. If you use libraries, be aware that an actual external function call is performed.
``msg.sender``, ``msg.value`` and ``this`` will retain their values in this call, though (prior to Homestead, because of the use of ``CALLCODE``, ``msg.sender`` and ``msg.value`` changed, though).

The following example shows how to use :ref:`types stored in memory <data-location>` and internal functions in libraries in order to implement custom types without the overhead of external function calls:

::

    pragma solidity >=0.4.16 <0.6.0;

    library BigInt {
        struct bigint {
            uint[] limbs;
        }

        function fromUint(uint x) internal pure returns (bigint memory r) {
            r.limbs = new uint[](1);
            r.limbs[0] = x;
        }

        function add(bigint memory _a, bigint memory _b) internal pure returns (bigint memory r) {
            r.limbs = new uint[](max(_a.limbs.length, _b.limbs.length));
            uint carry = 0;
            for (uint i = 0; i < r.limbs.length; ++i) {
                uint a = limb(_a, i);
                uint b = limb(_b, i);
                r.limbs[i] = a + b + carry;
                if (a + b < a || (a + b == uint(-1) && carry > 0))
                    carry = 1;
                else
                    carry = 0;
            }
            if (carry > 0) {
                // too bad, we have to add a limb
                uint[] memory newLimbs = new uint[](r.limbs.length + 1);
                uint i;
                for (i = 0; i < r.limbs.length; ++i)
                    newLimbs[i] = r.limbs[i];
                newLimbs[i] = carry;
                r.limbs = newLimbs;
            }
        }

        function limb(bigint memory _a, uint _limb) internal pure returns (uint) {
            return _limb < _a.limbs.length ? _a.limbs[_limb] : 0;
        }

        function max(uint a, uint b) private pure returns (uint) {
            return a > b ? a : b;
        }
    }

    contract C {
        using BigInt for BigInt.bigint;

        function f() public pure {
            BigInt.bigint memory x = BigInt.fromUint(7);
            BigInt.bigint memory y = BigInt.fromUint(uint(-1));
            BigInt.bigint memory z = x.add(y);
            assert(z.limb(1) > 0);
        }
    }

As the compiler cannot know where the library will be deployed at, these addresses have to be filled into the final bytecode by a linker (see :ref:`commandline-compiler` for how to use the commandline compiler for linking). If the addresses are not
given as arguments to the compiler, the compiled hex code will contain placeholders of the form ``__Set______`` (where
``Set`` is the name of the library). The address can be filled manually by replacing all those 40 symbols by the hex
encoding of the address of the library contract.

.. note::
    Manually linking libraries on the generated bytecode is discouraged, because it is restricted to 36 characters.
    You should ask the compiler to link the libraries at the time a contract is compiled by either using the ``--libraries`` option of ``solc`` or the ``libraries`` key if you use the standard-JSON interface to the compiler.

Restrictions for libraries in comparison to contracts:

- No state variables
- Cannot inherit nor be inherited
- Cannot receive Ether

(These might be lifted at a later point.)

.. _call-protection:

Call Protection For Libraries
=============================

As mentioned in the introduction, if a library's code is executed using a ``CALL`` instead of a ``DELEGATECALL`` or ``CALLCODE``, it will revert unless a ``view`` or ``pure`` function is called.

The EVM does not provide a direct way for a contract to detect whether it was called using ``CALL`` or not, but a contract
can use the ``ADDRESS`` opcode to find out "where" it is currently running. The generated code compares this address to the address used at construction time to determine the mode of calling.

More specifically, the runtime code of a library always starts with a push instruction, which is a zero of 20 bytes at
compilation time. When the deploy code runs, this constant is replaced in memory by the current address and this modified code is stored in the contract. At runtime, this causes the deploy time address to be the first constant to be pushed onto the stack and the dispatcher code compares the current address against this constant for any non-view and non-pure function.

.. index:: ! using for, library

.. _using-for:

*********
Using For
*********

The directive ``using A for B;`` can be used to attach library functions (from the library ``A``) to any type (``B``).
These functions will receive the object they are called on as their first parameter (like the ``self`` variable in Python).

The effect of ``using A for *;`` is that the functions from the library ``A`` are attached to *any* type.

In both situations, *all* functions in the library are attached, even those where the type of the first parameter does not
match the type of the object. The type is checked at the point the function is called and function overload resolution is performed.

The ``using A for B;`` directive is active only within the current contract, including within all of its functions, and has no effect outside of the contract in which it is used. The directive may only be used inside a contract, not inside any of its functions.

By including a library, its data types including library functions are available without having to add further code.

Let us rewrite the set example from the :ref:`libraries` in this way::

    pragma solidity >=0.4.16 <0.6.0;

    // This is the same code as before, just without comments
    library Set {
      struct Data { mapping(uint => bool) flags; }

      function insert(Data storage self, uint value)
          public
          returns (bool)
      {
          if (self.flags[value])
            return false; // already there
          self.flags[value] = true;
          return true;
      }

      function remove(Data storage self, uint value)
          public
          returns (bool)
      {
          if (!self.flags[value])
              return false; // not there
          self.flags[value] = false;
          return true;
      }

      function contains(Data storage self, uint value)
          public
          view
          returns (bool)
      {
          return self.flags[value];
      }
    }

    contract C {
        using Set for Set.Data; // this is the crucial change
        Set.Data knownValues;

        function register(uint value) public {
            // Here, all variables of type Set.Data have corresponding member functions.
            // The following function call is identical to `Set.insert(knownValues, value)`
            require(knownValues.insert(value));
        }
    }

It is also possible to extend elementary types in that way::

    pragma solidity >=0.4.16 <0.6.0;

    library Search {
        function indexOf(uint[] storage self, uint value)
            public
            view
            returns (uint)
        {
            for (uint i = 0; i < self.length; i++)
                if (self[i] == value) return i;
            return uint(-1);
        }
    }

    contract C {
        using Search for uint[];
        uint[] data;

        function append(uint value) public {
            data.push(value);
        }

        function replace(uint _old, uint _new) public {
            // This performs the library function call
            uint index = data.indexOf(_old);
            if (index == uint(-1))
                data.push(_new);
            else
                data[index] = _new;
        }
    }

Note that all library calls are actual EVM function calls. This means that if you pass memory or value types, a copy will be performed, even of the ``self`` variable. The only situation where no copy will be performed is when storage reference variables are used.
