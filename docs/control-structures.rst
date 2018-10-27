#####################################
Expressions et structures de contrôle
#####################################

.. index:: ! parameter, parameter;input, parameter;output

Paramètres d'entrée et de sortie
================================

Comme en Javascript, les fonctions peuvent prendre des paramètres en entrée; contrairement à Javascript et C, elles peuvent retourner plusieurs paramètres en sortie.

Paramètres d'entrée
-------------------

Les paramètres d'entrée sont déclarés de la même manière que les variables.
Le nom des paramètres non utilisés peut être omis.
Par exemple, supposons que nous voulions que notre contrat accepte un type d'appels externes avec deux entiers, nous pourrions écrire quelque chose comme::

    pragma solidity >=0.4.16 <0.6.0;

    contract Simple {
        uint sum;
        function taker(uint _a, uint _b) public {
            sum = _a + _b;
        }
    }

Les paramètres d'entrée peuvent être utilisés comme n'importe quelle autre variable locale, ils peuvent aussi être assignés.

Paramètres de sortie
--------------------

Les paramètres de sortie peuvent être déclarés avec la même syntaxe après le mot-clé ``returns``. Par exemple, supposons que nous voulions retourner deux résultats: la somme et le produit des deux entiers donnés, alors nous pourrions écrire::

    pragma solidity >=0.4.16 <0.6.0;

    contract Simple {
        function arithmetic(uint _a, uint _b)
            public
            pure
            returns (uint o_sum, uint o_product)
        {
            o_sum = _a + _b;
            o_product = _a * _b;
        }
    }

Les noms des paramètres de sortie peuvent être omis.
Les valeurs de sortie peuvent également être spécifiées à l'aide de l'instruction ``returns``, également capables de :ref:`return multiple values<multi-return>`.
Les paramètres de retour peuvent être utilisés comme n'importe quelle autre variable locale et sont initialisés zéro; s'ils ne sont pas explicitement définis, ils restent à zéro.

.. index:: if, else, while, do/while, for, break, continue, return, switch, goto

Structures de controle
======================

La plupart des structures de contrôle connues des langages à accolades sont disponibles dans Solidity :

Nous disposons de: ``if``, ``else``, ``while``, ``do``, ``for``, ``break``, ``continue``, ``return``, avec la syntaxe famillière du C ou du JavaScript.

Les parenthèses ne peuvent *pas* être omises pour les conditions, mais les accolades peuvent être omises autour des déclaration en une opération.

Notez qu'il n'y a pas de conversion de types non booléens vers types booléens comme en C et JavaScript, donc ``if (1) {...}`` n'est pas valable en Solidity.

.. _multi-return:

Retour de valeurs multiples
---------------------------

Lorsqu'une fonction a plusieurs paramètres de sortie, ``return (v0, v1, ...., vn)`` peut retourner plusieurs valeurs. Le nombre de composants doit être le même que le nombre de paramètres de sortie déclarés.

.. index:: ! function;call, function;internal, function;external

.. _function-calls:

Appels de fonction
==================

Appels de fonction internes
---------------------------

Les fonctions du contrat en cours peuvent être appelées directement (``ìnternal``), également de manière récursive, comme le montre cet exemple absurde::

    pragma solidity >=0.4.16 <0.6.0;

    contract C {
        function g(uint a) public pure returns (uint ret) { return a + f(); }
        function f() internal pure returns (uint ret) { return g(7) + f(); }
    }

Ces appels de fonction sont traduits en simples sauts( ``JUMP``) à l'intérieur de l'EVM. Cela a pour effet que la mémoire actuelle n'est pas effacée, c'est-à-dire qu'il est très efficace de passer des références de mémoire aux fonctions appelées en interne. Seules les fonctions du même contrat peuvent être appelées en interne.

Vous devriez toujours éviter une récursivité excessive, car chaque appel de fonction interne utilise au moins un emplacement de pile et il y a au maximum un peu moins de 1024 emplacements disponibles.

Appels de fonction externes
---------------------------

Les expressions ``this.g(8);`` et ``c.g(2);`` (où ``c`` est une instance de contrat) sont aussi des appels de fonction valides, mais cette fois-ci, la fonction sera appelée ``external``, via un appel de message et non directement via des sauts.
Veuillez noter que les appels de fonction sur ``this`` ne peuvent pas être utilisés dans le constructeur, car le contrat actuel n'a pas encore été créé.

Les fonctions d'autres contrats doivent être appelées en externe. Pour un appel externe, tous les arguments de fonction doivent être copiés en mémoire.

Lors de l'appel de fonctions d'autres contrats, le montant de Wei envoyé avec l'appel et le gas peut être spécifié avec les options spéciales ``.value()`` et ``.gas()``, respectivement::

    pragma solidity >=0.4.0 <0.6.0;

    contract InfoFeed {
        function info() public payable returns (uint ret) { return 42; }
    }

    contract Consumer {
        InfoFeed feed;
        function setFeed(InfoFeed addr) public { feed = addr; }
        function callFeed() public { feed.info.value(10).gas(800)(); }
    }

Vous devez utiliser le modificateur ``payable`` avec la fonction ``info`` pour pouvoir appeler ``.value()`` .

.. warning::
  Veillez à ce que ``feed.info.value(10).gas(800)`` ne définisse que localement la ``value`` et la quantité de ``gas`` envoyés avec l'appel de fonction, et que les parenthèses à la fin sont bien présentes pour effectuer l'appel. Ainsi, dans cet exemple, la fonction n'est pas appelée.

Les appels de fonction provoquent des exceptions si le contrat appelé n'existe pas (dans le sens où le compte ne contient pas de code) ou si le contrat appelé lui-même lève une exception ou manque de gas.

.. warning::
 Toute interaction avec un autre contrat présente un danger potentiel, surtout si le code source du contrat n'est pas connu à l'avance. Le contrat actuel cède le contrôle au contrat appelé et cela peut potentiellement faire à peu près n'importe quoi. Même si le contrat appelé hérite d'un contrat parent connu, le contrat d'héritage doit seulement avoir une interface correcte. L'exécution du contrat peut cependant être totalement arbitraire et donc représentent un danger. En outre, soyez prêt au cas où il appelle d'autres fonctions de votre contrat ou même de retour dans le contrat d'appel avant le retour du premier appel. Cela signifie que le contrat appelé peut modifier les variables d'état du contrat appelant via ses fonctions. Écrivez vos fonctions de manière à ce que, par exemple, les appels à
 les fonctions externes se produisent après tout changement de variables d'état dans votre contrat, de sorte que votre contrat n'est pas vulnérable à un exploit de réentrée.

Appels nommés et paramètres de fonction anonymes
------------------------------------------------

Les arguments d'appel de fonction peuvent être donnés par leur nom, dans n'importe quel ordre, s'ils sont inclus dans ``{ }`` comme on peut le voir dans l'exemple qui suit. La liste d'arguments doit coïncider par son nom avec la liste des paramètres de la déclaration de fonction, mais peut être dans un ordre arbitraire.

::

    pragma solidity >=0.4.0 <0.6.0;

    contract C {
        mapping(uint => uint) data;

        function f() public {
            set({value: 2, key: 3});
        }

        function set(uint key, uint value) public {
            data[key] = value;
        }

    }

Noms des paramètres de fonction omis
------------------------------------

Les noms des paramètres inutilisés (en particulier les paramètres de retour) peuvent être omis.
Ces paramètres seront toujours présents sur la pile, mais ils sont inaccessibles.

::

    pragma solidity >=0.4.16 <0.6.0;

    contract C {
        // omitted name for parameter
        function func(uint k, uint) public pure returns(uint) {
            return k;
        }
    }


.. index:: ! new, contracts;creating

.. _creating-contracts:

Création de contrats via ``new``
================================

Un contrat peut créer d'autres contrats en utilisant le mot-clé ``new``. Le code complet du contrat en cours de création doit être connu lors de la compilation afin d'éviter les dépendances récursives liées à la création.


::

    pragma solidity >0.4.99 <0.6.0;

    contract D {
        uint public x;
        constructor(uint a) public payable {
            x = a;
        }
    }

    contract C {
        D d = new D(4); // sera exécuté dans le constructor de C

        function createD(uint arg) public {
            D newD = new D(arg);
            newD.x();
        }

        function createAndEndowD(uint arg, uint amount) public payable {
            // Envoyer des Ethers avec la création
            D newD = (new D).value(amount)(arg);
            newD.x();
        }
    }

Comme dans l'exemple, il est possible d'envoyer des Ether en créant une instance de ``D`` en utilisant l'option ``.value()``, mais il n'est pas possible de limiter la quantité de gas.
Si la création échoue (à cause d'une rupture de pile, d'un manque de gas ou d'autres problèmes), une exception est levée.

Ordre d'évaluation des expressions
==================================

L'ordre d'évaluation des expressions n'est pas spécifié (plus formellement, l'ordre dans lequel les enfants d'un noeud de l'arbre des expressions sont évalués n'est pas spécifié, mais ils sont bien sûr évalués avant le noeud lui-même). La seule garantie est que les instructions sont exécutées dans l'ordre et que les expressions booléennes sont court-circuitées correctement. Voir :ref:`order` pour plus d'informations.

.. index:: ! assignment

Assignation
===========

.. index:: ! assignment;destructuring

Déstructuration d'assignations et retour de valeurs multiples
-------------------------------------------------------------

Solidity permet en interne les tuples, c'est-à-dire une liste d'objets de types potentiellement différents dont le nombre est une constante au moment de la compilation. Ces tuples peuvent être utilisés pour retourner plusieurs valeurs en même temps.
Ceux-ci peuvent ensuite être affectés soit à des variables nouvellement déclarées, soit à des variables préexistantes (ou à des LValues en général).

Les tuples ne sont pas des types propres à Solidity, ils ne peuvent être utilisés que pour former des groupes syntaxiques d'expressions.

::

    pragma solidity >0.4.23 <0.6.0;

    contract C {
        uint[] data;

        function f() public pure returns (uint, bool, uint) {
            return (7, true, 2);
        }

        function g() public {
            // Variables declared with type and assigned from the returned tuple,
            // not all elements have to be specified (but the number must match).
            (uint x, , uint y) = f();
            // Astuce simple pour un échange de valeurs -- Ne marche pas pour
            // les types autres que par valeur (voir Types).
            (x, y) = (y, x);
            // Certains composants peuvent être ignorés au besoin
            (data.length, , ) = f(); // Sets the length to 7
        }
    }

Il n'est pas possible de mélanger les assignations à la déclarations et les assignations simples, c'est-à-dire que ce qui suit n'est pas valable : `(x, uint y) = (1, 2);``

.. note::
    Avant la version 0.5.0, il était possible d'assigner des tuples de plus petite taille, soit en les remplissant à gauche ou à droite (ce qui était vide). Ceci est maintenant interdit, de sorte que les deux côtés doivent avoir le même nombre de composants, laissés blancs si inutilisés.

.. warning::
    Soyez prudent lorsque vous assignez plusieurs variables en même temps lorsqu'il s'agit de types de référence, car cela pourrait entraîner une copie inattendue.

Complications pour les tableaux et les structures
-------------------------------------------------

La sémantique des affectations est un peu plus compliquée pour les types autres que valeurs comme les tableaux et les structs.
L'affectation *à* une variable d'état crée toujours une copie indépendante. D'autre part, l'affectation à une variable locale crée une copie indépendante uniquement pour les types élémentaires, c'est-à-dire les types statiques qui tiennent sur 32 octets. Si des structs ou des tableaux (y compris les ``bytes`` et les ``string``) sont assignés d'une variable d'état à une variable locale, la variable locale contient une référence à la variable d'état originale. Une deuxième affectation à la variable locale ne modifie pas l'état mais seulement la référence. Les affectations aux membres (ou éléments) de la variable locale *changent* l'état.

.. index:: ! scoping, declarations, default value

.. _default-value:

Portée et déclarations
======================

Une variable qui est déclarée aura une valeur par défaut initiale dont la représentation octale est égale à une suite de zéros.
Les "valeurs par défaut" des variables sont les "états zéro" typiques quel que soit le type. Par exemple, la valeur par défaut d'un ``bool`` est ``false``. La valeur par défaut pour les types ``uint`` ou ``int`` est ``0``. Pour les tableaux de taille statique et les ``bytes1`` à ``bytes32``, chaque élément individuel sera initialisé à la valeur par défaut correspondant à son type. Enfin, pour les tableaux de taille dynamique, les octets et les chaînes de caractères, la valeur par défaut est un tableau ou une chaîne vide.

La portée en Solidity suit les règles de portée très répandues du C99 (et de nombreux autres languages): Les variables sont visibles du point situé juste après leur déclaration jusqu'à la fin du plus petit bloc ``{ }`` qui contient la déclaration. Par exception à cette règle, les variables déclarées dans la partie initialisation d'une boucle ``for`` ne sont visibles que jusqu'à la fin de la boucle for.

Les variables et autres éléments déclarés en dehors d'un bloc de code, par exemple les fonctions, les contrats, les types définis par l'utilisateur, etc. sont visibles avant même leur déclaration. Cela signifie que vous pouvez utiliser les variables d'état avant qu'elles ne soient déclarées et appeler les fonctions de manière récursive.

Par conséquent, les exemples suivants seront compilés sans avertissement, puisque les deux variables ont le même nom mais des portées disjointes.

::

    pragma solidity >0.4.99 <0.6.0;
    contract C {
        function minimalScoping() pure public {
            {
                uint same;
                same = 1;
            }

            {
                uint same;
                same = 3;
            }
        }
    }

À titre d'exemple particulier des règles de détermination de la portée héritées du C99, notons que, dans ce qui suit, la première affectation à ``x`` affectera en fait la variable externe et non la variable interne. Dans tous les cas, vous obtiendrez un avertissement concernant cette double déclaration.

::

    pragma solidity >0.4.99 <0.6.0;
    // Ceci déclenche un warning
    contract C {
        function f() pure public returns (uint) {
            uint x = 1;
            {
                x = 2; // Ceci assigne à la valeur externe
                uint x;
            }
            return x; // x == 2
        }
    }

.. warning::
    Avant la version 0.5.0, Solidity suivait les mêmes règles de scoping que JavaScript, c'est-à-dire qu'une variable déclarée n'importe où dans une fonction était dans le champ d'application pour l'ensemble de la fonction, peu importe où elle était déclarée. L'exemple suivant montre un extrait de code qui compilait, mais conduit aujourd'hui à une erreur à partir de la version 0.5.0.

 ::

    pragma solidity >0.4.99 <0.6.0;
    // Ceci ne compile plus
    contract C {
        function f() pure public returns (uint) {
            x = 2;
            uint x;
            return x;
        }
    }

.. index:: ! exception, ! throw, ! assert, ! require, ! revert

.. _assert-and-require:

Gestion d'erreurs: Assert, Require, Revert et Exceptions
========================================================

Solidity utilise des exceptions qui restaurent l'état pour gérer les erreurs. Une telle exception annule toutes les modifications apportées à l'état de l'appel en cours (et de tous ses sous-appels) et signale également une erreur à l'appelant.
Les fonctions bien pratiques ``assert`` et ``require`` peuvent être utilisées pour vérifier les conditions et lancer une exception si la condition n'est pas remplie. La fonction ``assert`` ne doit être utilisée que pour tester les erreurs internes, et pour vérifier les invariants.
La fonction ``require`` doit être utilisée pour s'assurer que les conditions valides, telles que les entrées ou les variables d'état du contrat, sont remplies, ou pour valider les valeurs de retour des appels aux contrats externes.
S'ils sont utilisés correctement, les outils d'analyse peuvent évaluer votre contrat afin d'identifier les conditions et les appels de fonction qui parviendront à un échec d'``assert``. Un code fonctionnant correctement ne devrait jamais échouer un ``assert`` ; si cela se produit, il y a un bogue dans votre contrat que vous devriez corriger.

Il y a deux autres façons de déclencher des exceptions: La fonction ``revert`` peut être utilisée pour signaler une erreur et annuler l'appel en cours. Il est possible de fournir une chaîne de caractères contenant des détails sur l'erreur qui sera renvoyée à l'appelant.

.. note::
     Il y avait un mot-clé appelé ``throw`` avec la même sémantique que ``revert()`` qui était déprécié dans la version 0.4.13 et supprimé dans la version 0.5.0.

Lorsque des exceptions se produisent dans un sous-appel, elles "remontent à la surface" automatiquement (c'est-à-dire que les exceptions sont déclenchées en casacade). Les exceptions à cette règle sont ``send`` et les fonctions de bas niveau ``call``, ``delegatecall`` et ``staticcall``, qui retournent ``false`` comme première valeur de retour en cas d'exception au lieu de lancer une chaine d'exceptions.

.. warning::
    Les fonctions de bas niveau ``call``, ``delegatecall`` et ``staticcall`` renvoient ``true`` comme première valeur de retour si le compte appelé est inexistant, dû à la conception de l'EVM. L'existence doit être vérifiée avant l'appel si désiré.

Il n'est pas encore possible de réellement réagir aux exceptions.

Dans l'exemple suivant, vous pouvez voir comment ``require`` peut être utilisé pour vérifier facilement les conditions sur les entrées et comment ``assert`` peut être utilisé pour vérifier les erreurs internes. Notez que vous pouvez facultativement fournir une chaîne de message pour ``require``, mais pas pour ``assert``.

::

    pragma solidity >0.4.99 <0.6.0;

    contract Sharer {
        function sendHalf(address payable addr) public payable returns (uint balance) {
            require(msg.value % 2 == 0, "Even value required.");
            uint balanceBeforeTransfer = address(this).balance;
            addr.transfer(msg.value / 2);
            // Étant donné que le transfert prévoit une exception en cas d'échec et
            // qu'il ne peut pas être rappelé ici, il ne devrait pas y avoir moyen
            // pour nous d'avoir encore la moitié de l'argent.
            assert(address(this).balance == balanceBeforeTransfer - msg.value / 2);
            return address(this).balance;
        }
    }

Une exception de type ``assert`` est générée dans les situations suivantes:

#. Si vous accédez à un tableau avec un index trop grand ou négatif (par ex. ``x[i]`` où ``i >= x.length`` ou ``i < 0``).
#. Si vous accédez à une variable de longueur fixe ``bytesN`` à un indice trop grand ou négatif.
#. Si vous divisez ou modulez par zéro (par ex. ``5 / 0`` ou ``23 % 0``).
#. Si vous décalez d'un montant négatif.
#. Si vous convertissez une valeur trop grande ou négative en un type enum.
#. Si vous appelez une variable initialisée nulle de type fonction interne.
#. Si vous appelez ``assert`` avec un argument qui s'évalue à ``false``.

Une exception de type ``assert`` est générée dans les situations suivantes:

#. Appeler ``require`` avec un argument qui s'évalue à ``false``.
#. Si vous appelez une fonction via un appel de message mais qu'elle ne se termine pas correctement (c'est-à-dire qu'elle n'a plus de gas, qu'elle n'a pas de fonction correspondante ou qu'elle lance une exception elle-même), sauf lorsqu'une opération de bas niveau ``call``, ``send``, ``staticcall``, ``delegatecall`` ou ``callcode`` est utilisée. Les opérations de bas niveau ne lancent jamais d'exceptions mais indiquent les échecs en retournant ``false``.
#. Si vous créez un contrat en utilisant le mot-clé ``new`` mais que la création du contrat ne se termine pas correctement (voir ci-dessus pour la définition de "ne pas terminer correctement").
#. Si vous effectuez un appel de fonction externe ciblant un contrat qui ne contient aucun code.
#. Si votre contrat reçoit des Ether via une fonction publique sans modificateur ``payable`` (y compris le constructeur et la fonction par defaut).
#. Si votre contrat reçoit des Ether via une fonction de getter public.
#. Si un ``.transfer()`` échoue.

En interne, Solidity exécute une opération de retour en arrière (instruction ``0xfd``) pour une exception de type ``require`` et exécute une opération invalide (instruction ``0xfe``) pour lancer une exception de type ``assert``. Dans les deux cas, cela provoque lánnulation toutes les modifications apportées à l'état de l'EVM dans l'appel courant. La raison du retour en arrière est qu'il n'y a pas de moyen sûr de continuer l'exécution, parce qu'un effet attendu ne s'est pas produit. Parce que nous voulons conserver l'atomicité des transactions, la chose la plus sûre à faire est d'annuler tous les changements et de faire toute la transaction (ou au moins l'appel) sans effet. 

.. note::
    Les exceptions de type ``assert`` consomment tout le gas disponible pour l'appel, alors que les exceptions de type ``require`` ne consommeront pas de gaz à partir du lancement de Metropolis.

L'exemple suivant montre comment une chaîne d'erreurs peut être utilisée avec ``revert`` et ``require`` :


::

    pragma solidity >0.4.99 <0.6.0;

    contract VendingMachine {
        function buy(uint amount) public payable {
            if (amount > msg.value / 2 ether)
                revert("Not enough Ether provided.");
            // Autre façon de le faire:
            require(
                amount <= msg.value / 2 ether,
                "Not enough Ether provided."
            );
            // Effectuer l'achat
        }
    }

La chaîne fournie sera :ref:`abi-encoded <ABI>` comme si c'était un appel à une fonction ``Error(string)``.
Dans l'exemple ci-dessus, ``revert("Not enough Ether provided.");``` fera en sorte que les données hexadécimales suivantes soient définies comme données de retour d'erreur :

.. code::

    0x08c379a0                                                         // Selecteur de fonction pour Error(string)
    0x0000000000000000000000000000000000000000000000000000000000000020 // Décalage des données
    0x000000000000000000000000000000000000000000000000000000000000001a // Taille de la string
    0x4e6f7420656e6f7567682045746865722070726f76696465642e000000000000 // Données de la string
