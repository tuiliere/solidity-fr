.. index:: type

.. _types:

*****
Types
*****

Solidity est un langage à typage statique, ce qui signifie que le type de chaque variable (état et locale) doit être spécifié.
Solidity propose plusieurs types élémentaires qui peuvent être combinés pour former des types complexes.

De plus, les types peuvent interagir entre eux dans des expressions contenant des opérateurs. Pour une liste synthétique des différents opérateurs, voir :ref:`order`.

.. index:: ! value type, ! type;value

Types Valeur
============

Les types suivants sont également appelés types valeur parce que les variables de ces types sont toujours transmises par valeur, c'est-à-dire qu'elles sont toujours copiées lorsqu'elles sont utilisées comme arguments de fonction ou dans les affectations.

.. index:: ! bool, ! true, ! false

Booléens
--------

``bool``: Les valeurs possibles sont les constantes ``true`` et ``false``.

Opérateurs:

*  ``!`` (négation logique)
*  ``&&`` (conjonction logique, "and")
*  ``||`` (disjonction logique, "or")
*  ``==`` (égalité)
*  ``!=`` (inégalité)

Les opérateurs ``||`` et ``&&`` appliquent les règles communes de court-circuitage. Celà signifie que ``f(x) || g(y)``, if ``f(x)`` evalue comme ``true``, ``g(y)`` ne sera pas exécutée même si ses effets étaient attendus.

.. index:: ! uint, ! int, ! integer

Entiers
-------

``int`` / ``uint``: Entiers signés et non-signés de différentes tailles. Les mots-clé ``uint8`` à ``uint256`` par pas de ``8`` (entier non signé de 8 à 256 bits) et ``int8`` à ``int256``. ``uint`` et ``int`` sont des alias de ``uint256`` et ``int256``, respectivement.

Opérateurs:

* Comparaisons: ``<=``, ``<``, ``==``, ``!=``, ``>=``, ``>`` (retournent un ``bool``)
* Opérateurs binaires: ``&``, ``|``, ``^`` (ou exclusif binaire), ``~`` (négation binaire)
* Opérateurs de décalage: ``<<`` (décalage vers la gauche), ``>>`` (décalage vers la droite)
* Opérateurs arithmétiques: ``+``, ``-``, l' opérateur unaire ``-``, ``*``, ``/``, ``%`` (modulo), ``**`` (exponentiation)


Comparaisons
^^^^^^^^^^^^

La valeur d'une comparaison est celle obtenue en comparant la valeur entière.

Opérations binaires
^^^^^^^^^^^^^^^^^^^

Les opérations binaires sont effectuées sur la représentation du nombre par `complément à deux<https://fr.wikipedia.org/wiki/Compl%C3%A9ment_%C3%A0_deux>`.
Cela signifie que, par exemple, ``~int256(0) == int256(-1)``.

Décalages
^^^^^^^^^

Le résultat d'une opération de décalage a le type de l'opérande de gauche. L'expression ``x << y`` est équivalente à ``x * 2**y`` et, pour les entiers positifs, ``x >> y`` est équivalente à ``x / 2**y``. Pour un ``x`` négatif, ``x >> y`` équivaut à diviser par une puissance de ``2`` en arrondissant vers le bas (vers l'infini négatif).
Décaler d'un nombre négatif de bits déclenche une exception.

.. warning::
    Avant la version ``0.5.0.0``, un décalage vers la droite ``x >> y`` pour un ``x`` négatif était équivalent à ``x / 2**y``, c'est-à-dire que les décalages vers la droite étaient arrondis vers zéro plutôt que vers l'infini négatif.

Addition, Soustraction et Multiplication
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

L'addition, la soustraction et la multiplication ont la sémantique habituelle.
Ils utilisent également la représentation du complément de deux, ce qui signifie, par exemple, que ``uint256(0) - uint256(1) == 2**256 - 1``. Vous devez tenir compte de ces débordements ("overflows") pour la conception de contrats sûrs.

L'expression ``x`` équivaut à ``(T(0) - x)`` où ``T`` est le type de ``x``. Cela signifie que ``-x`` ne sera pas négatif si le type de ``x`` est un type entier non signé. De plus, ``x`` peut être positif si ``x`` est négatif. Il y a une autre mise en garde qui découle également de la représentation en compléments de deux::

    int x = -2**255;
    assert(-x == x);

Cela signifie que même si un nombre est négatif, vous ne pouvez pas supposer que sa négation sera positive.


Division
^^^^^^^^

Puisque le type du résultat d'une opération est toujours le type d'un des opérandes, la division sur les entiers donne toujours un entier.
Dans Solidity, la division s'arrondit vers zéro. Cela signifie que ``int256(-5) / int256(2) == int256(-2)``.

Notez qu'en revanche, la division sur les :ref:`littéraux<literals<rational_literals>` donne des valeurs fractionnaires de précision arbitraire.

.. note::
  La division par zéra cause un échec d'``assert``.

Modulo
^^^^^^

L'opération modulo ``a % n`` donne le reste ``r`` après la division de l'opérande ``a`` par l'opérande ``n``, où ``q = int(a / n)`` et ``r = a - (n * q)``. Cela signifie que modulo donne le même signe que son opérande gauche (ou zéro) et ``a % n == -(abs(a) % n)`` est valable pour un ``a`` négatif:

 * ``int256(5) % int256(2) == int256(1)``
 * ``int256(5) % int256(-2) == int256(1)``
 * ``int256(-5) % int256(2) == int256(-1)``
 * ``int256(-5) % int256(-2) == int256(-1)``

.. note::
  La division par zéra cause un échec d'``assert``.

Exponentiation
^^^^^^^^^^^^^^

l'exponentiation n'est disponible que p[our les types signés. Veillez à ce que les types que vous utilisez soient suffisamment grands pour conserver le résultat et vous préparer à un éventuel effet d'enroulage (wrapping/int overflow).

.. note::
  ``0**0`` est défini par l'EVM comme étant ``1``.

.. index:: ! ufixed, ! fixed, ! fixed point number

Nombre à virgule fixe
---------------------

.. warning::
    Les numéros à point fixe ne sont pas encore entièrement pris en charge par Solidity. Ils peuvent être déclarés, mais ne peuvent pas être affectés à ou de.

``fixed`` / ``ufixed``: Nombre `a virgule fixe signés et non-signés de taille variable. Les mots-clés ``ufixedMxN`` et ``fixedMxN``, où ``M`` représente le nombre de bits pris par le type et ``N`` représente combien de décimales sont disponibles. ``M`` doit être divisible par 8 et peut aller de 8 à 256 bits. ``N`` doit être compris entre 0 et 80, inclusivement.
``ufixed`` et ``fixed`` sont des alias pour ``ufixed128x18`` et ``fixed128x18``, respectivement.

Opérateurs:

* Comparaisons: ``<=``, ``<``, ``==``, ``!=``, ``>=``, ``>`` (évalue à ``bool``)
* Operateurs arithmétiques: ``+``, ``-``, l'opérateur unaire ``-``, ``*``, ``/``, ``%`` (modulo)

.. note::
    La principale différence entre les nombres à virgule flottante (``float``et ``double`` dans de nombreux langages, plus précisément les nombres IEEE 754) et les nombres à virgule fixe est que le nombre de bits utilisés pour l'entier et la partie fractionnaire (la partie après le point décimal) est flexible dans le premier, alors qu'il est strictement défini dans le second. Généralement, en virgule flottante, presque tout l'espace est utilisé pour représenter le nombre, alors que seul un petit nombre de bits définit où se trouve le point décimal.

.. index:: address, balance, send, call, callcode, delegatecall, staticcall, transfer

.. _address:

Adresses
--------

Le type d'adresse se décline en deux versions, qui sont en grande partie identiques :

 - ``address`` : Contient une valeur de 20 octets (taille d'une adresse Ethereum).
 - ``address payable`` : Même chose que "adresse", mais avec les membres additionnels ``transfert`` et ``envoi``.

L'idée derrière cette distinction est que l'``address payable`` est une adresse à laquelle vous pouvez envoyer de l'éther, alors qu'une simple ``address`` ne peut être envoyée de l'éther.

Conversions de type :

Les conversions implicites de ``address payable`` à ``address`` sont autorisées, tandis que les conversions de ``address`` à ``address payable`` ne sont pas possibles.
.. note::
    La seule façon d'effectuer une telle conversion est d'utiliser une conversion intermédiaire en ``uint160``.

Les :ref:`adresses littérales<address_literals<address_literals>` peuvent être implicitement converties en ``address payable``.

Les conversions explicites vers et à partir de ``address`` sont autorisées pour les entiers, les entiers littéraux, les ``bytes20`` et les types de contrats avec les réserves suivantes :
Les conversions sous la forme ``address payable(x)`` ne sont pas permises. Au lieu de cela, le résultat d'une conversion sous forme ``adresse(x)`` donne une ``address payable`` si ``x`` est un contrat disposant d'une fonction par défaut (``fallback``) ``payable``, ou si ``x`` est de type entier, bytes fixes, ou littéral.
Sinon, l'adresse obtenue sera de type ``address``.
Dans les fonctions de signature externes, ``address`` est utilisé à la fois pour le type ``address``et ``address payable``.

.. note::
    Il se peut fort bien que vous n'ayez pas à vous soucier de la distinction entre ``address`` et ``address payable`` et que vous utilisiez simplement ``address`` partout. Par exemple, si vous utilisez la fonction :ref:`withdrawal pattern<withdrawal_pattern>`, vous pouvez (et devriez) stocker l'adresse elle-même comme ``address``, parce que vous invoquez la fonction ``transfer`` sur
     ``msg.sender``, qui est une ``address payable``.

Opérateurs :

* ``<=``, ``<``, ``==``, ``!=``, ``>=`` and ``>``

.. warning::
    Si vous convertissez un type qui utilise une taille d'octet plus grande en ``address``, par exemple ``bytes32``, alors l'adresse est tronquée.
     Pour réduire l'ambiguïté de conversion à partir de la version 0.4.24 du compilateur vous force à rendre la troncature explicite dans la conversion.
     Prenons par exemple l'adresse ``0x1111222222323333434444545555666666777777778888999999AAAABBBBBBCCDDDDEEFEFFFFFFCC``.

     Vous pouvez utiliser ``address(uint160(octets20(b)))``, ce qui donne ``0x1111212222323333434444545555666677778888889999aAaaa``,
     ou vous pouvez utiliser ``address(uint160(uint256(b)))``, ce qui donne ``0x777777888888999999AaAAbBbbCcccddDdeeeEfFFfCcCcCc``.

.. note::
    La distinction entre ``address``et ``address payable`` a été introduite avec la version 0.5.0.
     À partir de cette version également, les contrats ne dérivent pas du type d'adresse, mais peuvent toujours être convertis explicitement en
     adresse " ou à " adresse payable ", s'ils ont une fonction par défaut payable.

.. _members-of-addresses:

Membres de Address
^^^^^^^^^^^^^^^^^^

Pour une liste des membres de address, voir :ref:`address_related`.

* ``balance`` et ``transfer``.

Il est possible d'interroger le solde d'une adresse en utilisant la propriété ``balance``
et d'envoyer des Ether (en unités de wei) à une adresse payable à l'aide de la fonction ``transfert`` :

::

    address payable x = address(0x123);
    address myAddress = address(this);
    if (x.balance < 10 && myAddress.balance >= 10) x.transfer(10);

La fonction ``transfer`` échoue si le solde du contrat en cours n'est pas suffisant ou si le transfert d'Ether est rejeté par le compte destinataire. La fonction ``transfert`` s'inverse en cas d'échec.

.. note::
    Si ``x`` est une adresse de contrat, son code (plus précisément : sa :ref:`fallback-function`, si présente) sera exécutée avec l'appel ``transfer`` (c'est une caractéristique de l'EVM et ne peut être empêché). Si cette exécution échoue ou s'il n'y a plus de gas, le transfert d'Ether sera annulé et le contrat en cours s'arrêtera avec une exception.

* ``send``

``send`` est la contrepartie de bas niveau du ``transfer``. Si l'exécution échoue, le contrat en cours ne s'arrêtera pas avec une exception, mais ``send`` retournera ``false``.

.. warning::
    Il y a certains dangers à utiliser la fonction ``send`` : Le transfert échoue si la profondeur de la stack atteint 1024 (cela peut toujours être forcé par l'appelant) et il échoue également si le destinataire manque de gas. Donc, afin d'effectuer des transferts d'Ether en toute sécurité, vérifiez toujours la valeur de retour de ``send``, utilisez ``transfer`` ou mieux encore  : utilisez un modèle où le destinataire retire l'argent.

* ``call``, ``delegatecall`` et ``staticcall``

Afin de s'interfacer avec des contrats qui ne respectent pas l'ABI, ou d'obtenir un contrôle plus direct sur l'encodage,
les fonctions ``call``, ``delegatecall`` et ``staticcall`` sont disponibles.
Elles prennent tous pour argument un seul ``bytes memory`` comme entrée et retournent la condition de succès (en tant que ``bool``) et les données (``bytes memory``).
Les fonctions ``abi.encoder``, ``abi.encoderPacked``, ``abi.encoderWithSelector`` et ``abi.encoderWithSignature`` peuvent être utilisées pour coder des données structurées.

Exemple::

    bytes memory payload = abi.encodeWithSignature("register(string)", "MyName");
    (bool success, bytes memory returnData) = address(nameReg).call(payload);
    require(success);

.. warning::
    Toutes ces fonctions sont des fonctions de bas niveau et doivent être utilisées avec précaution.
     Plus précisément, tout contrat inconnu peut être malveillant et si vous l'appelez, vous transférez le contrôle à ce contrat qui, à son tour, peut revenir dans votre contrat, donc soyez prêt à modifier les variables de votre état.
     quand l'appel revient. La façon habituelle d'interagir avec d'autres contrats est d'appeler une fonction sur un objet ``contract`` (``x.f()``)..

:: note::
    Les versions précédentes de Solidity permettaient à ces fonctions de recevoir des arguments arbitraires et de traiter différemment un premier argument de type ``bytes4``. Ces cas rares ont été supprimés dans la version 0.5.0.

Il est possible de régler le gas fourni avec le modificateur ``.gas()``::

    namReg.call.gas(1000000)(abi.encodeWithSignature("register(string)", "MyName"));

De même, la valeur en Ether fournie peut également être contrôlée: :::

    nameReg.call.value(1 ether)(abi.encodeWithSignature("register(string)", "MyName"));

Enfin, ces modificateurs peuvent être combinés. Leur ordre n'a pas d'importance::

    nameReg.call.gas(1000000).value(1 ether)(abi.encodeWithSignature("register(string)", "MyName"));

De la même manière, la fonction ``delegatecall`` peut être utilisée: la différence est que seul le code de l'adresse donnée est utilisé, tous les autres aspects (stockage, balance,...) sont repris du contrat actuel. Le but de ``delegatecall`` est d'utiliser du code de bibliothèque qui est stocké dans un autre contrat. L'utilisateur doit s'assurer que la disposition du stockage dans les deux contrats est adaptée à l'utilisation de ``delegatecall``.

.. note::
    Avant Homestead, il n'existait qu'une variante limitée appelée ``callcode`` qui ne donnait pas accès aux valeurs originales ``msg.sender`` et ``msg.value``. Cette fonction a été supprimée dans la version 0.5.0.

Depuis Byzantium, ``staticcall`` peut aussi être utilisé. C'est fondamentalement la même chose que ``call``, mais reviendra en arrière si la fonction appelée modifie l'état d'une manière ou d'une autre.

Les trois fonctions ``call``, ``delegatecall``et ``staticcall`` sont des fonctions de très bas niveau et ne devraient être utilisées qu'en *dernier recours* car elles brisent la sécurité de type de Solidity.

L'option ``.gas()`` est disponible sur les trois méthodes, tandis que l'option ``.value()`` n'est pas supportée pour ``delegatecall``.

.. note::
    Tous les contrats pouvant être convertis en type ``address``, il est possible d'interroger le solde du contrat en cours en utilisant ``address(this).balance``.

.. index:: ! contract type, ! type; contract

.. _contract_types:

Types Contrat
-------------

Chaque :ref:`contrat<contracts>` définit son propre type.
Vous pouvez implicitement convertir des contrats en contrats dont ils héritent.
Les contrats peuvent être explicitement convertis de et vers tous les autres types de contrats et le type ``address``.

La conversion explicite vers et depuis le type ``address payable`` n'est possible que si le type de contrat dispose d'une fonction de repli payante.
La conversion est toujours effectuée en utilisant ``address(x)`` et non ``address payable(x)``. Vous trouverez plus d'informations dans la section sur le :ref:`type address<address>`.

.. note::
     Avant la version 0.5.0, les contrats dérivaient directement du type address et il n'y avait aucune distinction entre ``address`` et ``address payable``.

Si vous déclarez une variable locale de type contrat (`MonContrat c`), vous pouvez appeler des fonctions sur ce contrat. Prenez bien soin de l'assigner à un contrat d'un type correspondant.

Vous pouvez également instancier les contrats (ce qui signifie qu'ils sont nouvellement créés). Vous trouverez plus de détails dans la section :ref:`'contrats de création'<contrats de création>`.

La représentation des données d'un contrat est identique à celle du type ``address`` et ce type est également utilisé dans l':ref:`ABI<ABI>`.

Les contrats ne supportent aucun opérateur.

Les membres du type contrat sont les fonctions externes du contrat, y compris les variables d'état publiques.

.. index:: byte array, bytes32

Tableaux d'octets de taille fixe
--------------------------------

Les types valeur ``bytes1``, ``bytes2``, ``bytes3``, ..., ``bytes32`` contiennent une séquence de 1 à 32 octets.
``byte`` est un alias de ``bytes1``.

Opérateurs:

* Comparaisons: ``<=``, ``<``, ``==``, ``!=``, ``>=``, ``>`` (retournent un ``bool``)
* Opérateurs binaires: ``&``, ``|``, ``^`` (ou exclusif binaire), ``~`` (négation binaire)
* Opérateurs de décalage: ``<<`` (décalage vers la gauche), ``>>`` (décalage vers la droite)
* Accès par indexage: Si ``x`` estd e type ``bytesI``, alors ``x[k]`` pour ``0 <= k < I`` retourne le ``k`` ème byte (lecture seule).

L'opérateur de décalage travaille avec n'importe quel type d'entier comme opérande droite (mais retourne le type de l'opérande gauche), qui indique le nombre de bits à décaler.
Le décalage d'un montant négatif entraîne une exception d'exécution.

Membres :

*``.length``` donne la longueur fixe du tableau d'octets (lecture seule).

.. note::
    Le type ``byte[]`` est un tableau d'octets, mais en raison des règles de bourrage, il gaspille 31 octets d'espace pour chaque élément (sauf en storage). Il est préférable d'utiliser le type "bytes" à la place.

Tableaux dynamiques d'octets
----------------------------

``bytes``:
    Tableau d'octets de taille dynamique, voir :ref:``arrays``. Ce n'est pas un type valeur !
``string``:
    Chaîne codée UTF-8 de taille dynamique, voir :ref:`arrays`. Ce n'est pas un type valeur !

.. index:: address, literal;address

.. _address_literals:

Adresses Littérales
-------------------

Les caractères hexadécimaux qui réussissent un test de somme de contrôle d'adresse ("address checksum"), par exemple ``0xdCad3a6d3569DF655070DEd06cb7A1b2Ccd1D3AF`` sont de type ``address payable``.
Les nombres hexadécimaux qui ont entre 39 et 41 chiffres et qui ne passent pas le test de somme de contrôle produisent un avertissement et sont traités comme des nombres rationnels littéraux réguliers.

.. note::
    Le format de some de contr^ole multi-casse est décrit dans `EIP-55 <https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md>`_.

.. index:: literal, literal;rational

.. _rational_literals:

Rationels et entiers littéraux
------------------------------

Les nombres entiers littéraux sont formés à partir d'une séquence de nombres compris entre 0 et 9 interprétés en décimal. Par exemple, ``69`` signifie soixante-neuf.
Les littéraux octaux n'existent pas dans Solidity et les zéros précédant un nombre sont invalides.

Les fractions décimales sont formées par un ``.`` avec au moins un chiffre sur un côté. Exemples : ``1.1``, ``.1 `` et ``1.3``.

La notation scientifique est également supportée, où la base peut avoir des fractions, alors que l'exposant ne le peut pas.
Exemples : ``2e10``, ``-2e10``, ``2e-10``, ``2e-10``, ``2.5e1``.

Les soulignements (underscore) peuvent être utilisés pour séparer les chiffres d'un nombre littéral numérique afin d'en faciliter la lecture.
Par exemple, la décimale ``123_000``, l'hexadécimale ``0x2eff_abde``, la notation décimale scientifique ``1_2e345_678`` sont toutes valables.
Les tirets de soulignement ne sont autorisés qu'entre deux chiffres et un seul tiret de soulignement consécutif est autorisé.
Il n'y a pas de signification sémantique supplémentaire ajoutée à un nombre contenant des tirets de soulignement, les tirets de soulignement sont ignorés.

Les expressions littérales numériques conservent une précision arbitraire jusqu'à ce qu'elles soient converties en un type non littéral (c'est-à-dire en les utilisant avec une expression non littérale ou par une conversion explicite).
Cela signifie que les calculs ne débordent pas (overflow) et que les divisions ne tronquent pas les expressions littérales des nombres.

Par exemple, ``(2**800 + 1) - 2**800`` produit la constante ``1`` (de type ``uint8``) bien que les résultats intermédiaires ne rentrent même pas dans la taille d'un mot machine. De plus, ``.5 * 8`` donne l'entier ``4`` (bien que des nombres non entiers aient été utilisés entre les deux).

N'importe quel opérateur qui peut être appliqué aux nombres entiers peut également être appliqué aux expressions littérales des nombres tant que les opérandes sont des nombres entiers. Si l'un des deux est fractionnaire, les opérations sur bits sont interdites et l'exponentiation est interdite si l'exposant est fractionnaire (parce que cela pourrait résulter en un nombre non rationnel).

.. note::
    Solidity a un type de nombre littéral pour chaque nombre rationnel.
     Les nombres entiers littéraux et les nombres rationnels appartiennent à des types de nombres littéraux.
     De plus, toutes les expressions numériques littérales (c'est-à-dire les expressions qui ne contiennent que des nombres et des opérateurs) appartiennent à des types littéraux de nombres. Ainsi, les expressions littérales ``1 + 2`` et ``2 + 1`` appartiennent toutes deux au même type littéral de nombre pour le nombre rationnel numéro trois.

.. warning::
    La dvision d'entiers littéraux tronquait dans les versions de Solidity avant la version 0.4.0, mais elle donne maintenant en un nombre rationnel, c'est-à-dire que ``5 / 2`` n'est pas égal à ``2``, mais à ``2.5``.

.. note::
    Les expressions littérales numériques sont converties en caractères non littéraux dès qu'elles sont utilisées avec des expressions non littérales. Indépendamment des types, la valeur de l'expression assignée à ``b`` ci-dessous est évaluée en entier. Comme ``a`` est de type ``uint128``, l'expression ``2,5 + a`` doit cependant avoir un type. Puisqu'il n'y a pas de type commun pour les types ``2.5`` et ``uint128``, le compilateur Solidity n'accepte pas ce code.

::

    uint128 a = 1;
    uint128 b = 2.5 + a + 0.5;

.. index:: literal, literal;string, string
.. _string_literals:

Chaines de caractères littérales
--------------------------------

Les chaînes de caractères littérales sont écrites avec des guillemets simples ou doubles (``"foo"`` ou ``'bar'``). Elles n'impliquent pas de zéro final comme en C ; ``foo`` représente trois octets, pas quatre. Comme pour les entiers littéraux, leur type peut varier, mais ils sont implicitement convertibles en ``bytes1``, ..., ``bytes32``, ou s'ils conviennent, en ``bytes`` et en ``string``.

Les chaînes de caractères littérales supportent les caractères d'échappement suivants :

 - ``\<newline>`` (échappe un réel caractère newline)
 - ``\\`` (barre oblique)
 - ``\'`` (guillemet simple)
 - ``\"`` (guillemet double)
 - ``\b`` (backspace)
 - ``\f`` (form feed)
 - ``\n`` (newline)
 - ``\r`` (carriage return)
 - ``\t`` (tabulation horizontale)
 - ``\v`` (tabulation verticale)
 - ``\xNN`` (hex escape, see below)
 - ``\uNNNN`` (echapement d'unicode, voir ci-dessous)

``\xNN`` prend une valeur hexadécimale et insère l'octet approprié, tandis que ``\uNNNNN`` prend un codepoint Unicode et insère une séquence UTF-8.

La chaîne de caractères de l'exemple suivant a une longueur de dix octets.
Elle commence par un octet de newline, suivi d'une guillemet double, d'une guillemet simple, d'un caractère barre oblique inversée et ensuite (sans séparateur) de la séquence de caractères ``abcdef``.

::

    "\n\"\'\\abc\
    def"

Tout terminateur de ligne unicode qui n'est pas une nouvelle ligne (i.e. LF, VF, FF, CR, NEL, LS, PS) est considéré comme terminant la chaîne littérale. Newline ne termine la chaîne littérale que si elle n'est pas précédée d'un ``\``.

.. index:: literal, bytes

Hexadécimaux littéraux
----------------------

Les caractères hexadécimaux sont précédées du mot-clé ``hex`` et sont entourées de guillemets simples ou doubles (``hex"001122FF"``). Leur contenu doit être une chaîne hexadécimale et leur valeur sera la représentation binaire de ces valeurs.

Les littéraux hexadécimaux se comportent comme :ref:`chaînes de caractères littérales<string_literals>` et ont les mêmes restrictions de convertibilité.

.. index:: enum

.. _enums:

Énumérations
------------

Les ``enum`` sont une façon de créer un type défini par l'utilisateur en Solidity. Ils sont explicitement convertibles de et vers tous les types d'entiers mais la conversion implicite n'est pas autorisée. La conversion explicite à partir d'un nombre entier vérifie au moment de l'exécution que la valeur se trouve à l'intérieur de la plage de l'enum et provoque une affirmation d'échec autrement.
Un enum a besoin d'au moins un membre.

La représentation des données est la même que pour les énumérations en C : Les options sont représentées par des valeurs entières non signées à partir de ``0``.


::

    pragma solidity >=0.4.16 <0.6.0;

    contract test {
        enum ActionChoices { GoLeft, GoRight, GoStraight, SitStill }
        ActionChoices choice;
        ActionChoices constant defaultChoice = ActionChoices.GoStraight;

        function setGoStraight() public {
            choice = ActionChoices.GoStraight;
        }

        // Comme le type enum ne fait pas partie de l' ABI, la signature de "getChoice"
        // sera automatoquement changée en "getChoice() returns (uint8)"
        // pour ce qui sort de Solidity. Le type entier utilisé est
        // assez grand pour contenir toutes valeurs, par exemple si vous en avez
        // plus de 256, ``uint16`` sera utilisé etc...
        function getChoice() public view returns (ActionChoices) {
            return choice;
        }

        function getDefaultChoice() public pure returns (uint) {
            return uint(defaultChoice);
        }
    }

.. index:: ! function type, ! type; function

.. _function_types:

Types Fonction
--------------

Les types fonction sont les types des fonctions. Les variables du type fonction peuvent être passés et retournés pour transférer les fonctions vers et renvoyer les fonctions des appels de fonction.
Les types de fonctions se déclinent en deux versions : les fonctions *internes* ``internal`` et les fonctions *externes* ``external`` :

Les fonctions internes ne peuvent être appelées qu'à l'intérieur du contrat en cours (plus précisément, à l'intérieur de l'unité de code en cours, qui comprend également les fonctions de bibliothèque internes et les fonctions héritées) car elles ne peuvent pas être exécutées en dehors du contexte du contrat actuel. L'appel d'une fonction interne est réalisé en sautant à son label d'entrée, tout comme lors de l'appel interne d'une fonction du contrat en cours.

Les fonctions externes se composent d'une adresse et d'une signature de fonction et peuvent être transférées et renvoyées à partir des appels de fonction externes.

Les types de fonctions sont notés comme suit: :

     fonction (<types de paramètres>) {internal|external} {pure|view|payable][returns (<types de retour>)]

En contraste avec types de paramètres, les types de retour ne peuvent pas être vides - si le type de fonction ne retourne rien, toute la partie ``returns (<types de retour>)``doit être omise.

Par défaut, les fonctions sont de type ``internal``, donc le mot-clé ``internal`` peut être omis. Notez que ceci ne s'applique qu'aux types de fonctions. La visibilité doit être spécifiée explicitement car les fonctions définies dans les contrats n'ont pas de valeur par défaut.

Conversions :

Une fonction de type ``external`` peut être explicitement convertie en ``address`` résultant en l'adresse du contrat de la fonction.

Un type de fonction ``A`` est implicitement convertible en un type de fonction ``B`` si et seulement si leurs types de paramètres sont identiques, leurs types de retour sont identiques, leurs propriétés internal/external sont identiques et la mutabilité d'état de ``A`` n'est pas plus restrictive que la mutabilité de l'état de ``B``. En particulier :

 - Les fonctions ``pure`` peuvent être converties en fonctions ``view`` et ``non-payable``.
 - Les fonctions ``view`` peuvent être converties en fonctions ``non-payable``.
 - les fonctions ``payable`` peuvent être converties en fonctions ``non-payable``.

Aucune autre conversion entre les types de fonction n'est possible.

La règle concernant les fonctions ``payable`` et ``non-payable`` peut prêter à confusion, mais essentiellement, si une fonction est ``payable``, cela signifie qu'elle accepte aussi un paiement de zéro Ether, donc elle est également ``non-payable``.
D'autre part, une fonction ``non-payable`` rejettera l'Ether qui lui est envoyé, de sorte que les fonctions ``non-payable`` ne peuvent pas être converties en fonctions ``payable``.

Si une variable de type fonction n'est pas initialisée, l'appel de celle-ci entraîne l'échec d'une assertion. Il en va de même si vous appelez une fonction après avoir utilisé ``delete`` dessus.

Si des fonctions de type ``external`` sont appelées d'en dehors du contexte de Solidity, ils sont traités comme le type ``function``, qui code l'adresse suivie de l'identificateur de fonction ensemble dans un seul type ``bytes24``.

Notez que les fonctions publiques du contrat actuel peuvent être utilisées à la fois comme une fonction interne et comme une fonction externe. Pour utiliser ``f`` comme fonction interne, utilisez simplement ``f``, si vous voulez utiliser sa forme externe, utilisez ``this.f```.

Membres :

Les fonctions publiques (ou externes) ont aussi un membre spécial appelé ``selector``, qui retourne le :ref:`sélecteur de fonction <abi_function_selector>`::

    pragma solidity >=0.4.16 <0.6.0;

    contract Selector {
      function f() public pure returns (bytes4) {
        return this.f.selector;
      }
    }

Example that shows how to use internal function types::

    pragma solidity >=0.4.16 <0.6.0;

    library ArrayUtils {
      // internal functions can be used in internal library functions because
      // they will be part of the same code context
      function map(uint[] memory self, function (uint) pure returns (uint) f)
        internal
        pure
        returns (uint[] memory r)
      {
        r = new uint[](self.length);
        for (uint i = 0; i < self.length; i++) {
          r[i] = f(self[i]);
        }
      }
      function reduce(
        uint[] memory self,
        function (uint, uint) pure returns (uint) f
      )
        internal
        pure
        returns (uint r)
      {
        r = self[0];
        for (uint i = 1; i < self.length; i++) {
          r = f(r, self[i]);
        }
      }
      function range(uint length) internal pure returns (uint[] memory r) {
        r = new uint[](length);
        for (uint i = 0; i < r.length; i++) {
          r[i] = i;
        }
      }
    }

    contract Pyramid {
      using ArrayUtils for *;
      function pyramid(uint l) public pure returns (uint) {
        return ArrayUtils.range(l).map(square).reduce(sum);
      }
      function square(uint x) internal pure returns (uint) {
        return x * x;
      }
      function sum(uint x, uint y) internal pure returns (uint) {
        return x + y;
      }
    }

Another example that uses external function types::

    pragma solidity >=0.4.22 <0.6.0;

    contract Oracle {
      struct Request {
        bytes data;
        function(uint) external callback;
      }
      Request[] requests;
      event NewRequest(uint);
      function query(bytes memory data, function(uint) external callback) public {
        requests.push(Request(data, callback));
        emit NewRequest(requests.length - 1);
      }
      function reply(uint requestID, uint response) public {
        // Here goes the check that the reply comes from a trusted source
        requests[requestID].callback(response);
      }
    }

    contract OracleUser {
      Oracle constant oracle = Oracle(0x1234567); // known contract
      uint exchangeRate;
      function buySomething() public {
        oracle.query("USD", this.oracleResponse);
      }
      function oracleResponse(uint response) public {
        require(
            msg.sender == address(oracle),
            "Only oracle can call this."
        );
        exchangeRate = response;
      }
    }

.. note::
    Lambda or inline functions are planned but not yet supported.

.. index:: ! type;reference, ! reference type, storage, memory, location, array, struct

Reference Types
===============

Values of reference type can be modified through multiple different names.
Contrast this with value types where you get an independent copy whenever a variable of value type is used. Because of that, reference types have to be handled more carefully than value types. Currently, reference types comprise structs, arrays and mappings. If you use a reference type, you always have to explicitly provide the data area where the type is stored: ``memory`` (whose lifetime is limited to a function call), ``storage`` (the location where the state variables are stored) or ``calldata`` (special data location that contains the function arguments, only available for external function call parameters).

An assignment or type conversion that changes the data location will always incur an automatic copy operation, while assignments inside the same data location only copy in some cases for storage types.

.. _data-location:

Data location
-------------

Every reference type, i.e. *arrays* and *structs*, has an additional annotation, the "data location", about where it is stored. There are three data locations:
``memory``, ``storage`` and ``calldata``. Calldata is only valid for parameters of external contract functions and is required for this type of parameter. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.


.. note::
    Prior to version 0.5.0 the data location could be omitted, and would default to different locations depending on the kind of variable, function type, etc., but all complex types must now give an explicit data location.

Data locations are not only relevant for persistency of data, but also for the semantics of assignments:
assignments between storage and memory (or from calldata) always create an independent copy.
Assignments from memory to memory only create references. This means that changes to one memory variable are also visible in all other memory variables that refer to the same data.
Assignments from storage to a local storage variables also only assign a reference.
In contrast, all other assignments to storage always copy. Examples for this case are assignments to state variables or to members of local variables of storage struct type, even if the local variable itself is just a reference.

::

    pragma solidity >=0.4.0 <0.6.0;

    contract C {
        uint[] x; // the data location of x is storage

        // the data location of memoryArray is memory
        function f(uint[] memory memoryArray) public {
            x = memoryArray; // works, copies the whole array to storage
            uint[] storage y = x; // works, assigns a pointer, data location of y is storage
            y[7]; // fine, returns the 8th element
            y.length = 2; // fine, modifies x through y
            delete x; // fine, clears the array, also modifies y
            // The following does not work; it would need to create a new temporary /
            // unnamed array in storage, but storage is "statically" allocated:
            // y = memoryArray;
            // This does not work either, since it would "reset" the pointer, but there
            // is no sensible location it could point to.
            // delete y;
            g(x); // calls g, handing over a reference to x
            h(x); // calls h and creates an independent, temporary copy in memory
        }

        function g(uint[] storage) internal pure {}
        function h(uint[] memory) public pure {}
    }

.. index:: ! array

.. _arrays:

Arrays
------

Arrays can have a compile-time fixed size or they can be dynamic.
The are few restrictions for the element, it can also be another array, a mapping or a struct. The general restrictions for
types apply, though, in that mappings can only be used in storage and publicly-visible functions need parameters that are ABI types.

An array of fixed size ``k`` and element type ``T`` is written as ``T[k]``, an array of dynamic size as ``T[]``. As an example, an array of 5 dynamic arrays of ``uint`` is ``uint[][5]`` (note that the notation is reversed when compared to some other languages). To access the second uint in the third dynamic array, you use ``x[2][1]`` (indices are zero-based and
access works in the opposite way of the declaration, i.e. ``x[2]`` shaves off one level in the type from the right).

Accessing an array past its end causes a revert. If you want to add new elements, you have to use ``.push()`` or increase the ``.length`` member (see below).

Variables of type ``bytes`` and ``string`` are special arrays. A ``bytes`` is similar to ``byte[]``, but it is packed tightly in calldata and memory. ``string`` is equal to ``bytes`` but does not allow length or index access.
So ``bytes`` should always be preferred over ``byte[]`` because it is cheaper.
As a rule of thumb, use ``bytes`` for arbitrary-length raw byte data and ``string`` for arbitrary-length string (UTF-8) data. If you can limit the length to a certain number of bytes, always use one of ``bytes1`` to ``bytes32`` because they are much cheaper.

.. note::
    If you want to access the byte-representation of a string ``s``, use ``bytes(s).length`` / ``bytes(s)[7] = 'x';``. Keep in mind that you are accessing the low-level bytes of the UTF-8 representation, and not the individual characters!

It is possible to mark arrays ``public`` and have Solidity create a :ref:`getter <visibility-and-getters>`.
The numeric index will become a required parameter for the getter.

.. index:: ! array;allocating, new

Allocating Memory Arrays
^^^^^^^^^^^^^^^^^^^^^^^^

You can use the ``new`` keyword to create arrays with a runtime-dependent length in memory.
As opposed to storage arrays, it is **not** possible to resize memory arrays (e.g. by assigning to the ``.length`` member). You either have to calculate the required size in advance or create a new memory array and copy every element.

::

    pragma solidity >=0.4.16 <0.6.0;

    contract C {
        function f(uint len) public pure {
            uint[] memory a = new uint[](7);
            bytes memory b = new bytes(len);
            assert(a.length == 7);
            assert(b.length == len);
            a[6] = 8;
        }
    }

.. index:: ! array;literals, !inline;arrays

Array Literals / Inline Arrays
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Array literals are arrays that are written as an expression and are not assigned to a variable right away.

::

    pragma solidity >=0.4.16 <0.6.0;

    contract C {
        function f() public pure {
            g([uint(1), 2, 3]);
        }
        function g(uint[3] memory) public pure {
            // ...
        }
    }

The type of an array literal is a memory array of fixed size whose base type is the common type of the given elements. The type of ``[1, 2, 3]`` is ``uint8[3] memory``, because the type of each of these constants is ``uint8``.
Because of that, it is necessary to convert the first element in the example above to ``uint``. Note that currently, fixed size memory arrays cannot be assigned to dynamically-sized memory arrays, i.e. the following is not
possible:

::

    pragma solidity >=0.4.0 <0.6.0;

    // This will not compile.
    contract C {
        function f() public {
            // The next line creates a type error because uint[3] memory
            // cannot be converted to uint[] memory.
            uint[] memory x = [uint(1), 3, 4];
        }
    }

It is planned to remove this restriction in the future but currently creates some complications because of how arrays are passed in the ABI.

.. index:: ! array;length, length, push, pop, !array;push, !array;pop

Members
^^^^^^^

**length**:
    Arrays have a ``length`` member that contains their number of elements.
    The length of memory arrays is fixed (but dynamic, i.e. it can depend on runtime parameters) once they are created.
    For dynamically-sized arrays (only available for storage), this member can be assigned to resize the array.
    Accessing elements outside the current length does not automatically resize the array and instead causes a failing assertion.
    Increasing the length adds new zero-initialised elements to the array.
    Reducing the length performs an implicit :ref:``delete`` on each of the removed elements.
**push**:
     Dynamic storage arrays and ``bytes`` (not ``string``) have a member function called ``push`` that you can use to append an element at the end of the array. The element will be zero-initialised. The function returns the new length.
**pop**:
     Dynamic storage arrays and ``bytes`` (not ``string``) have a member function called ``pop`` that you can use to remove an element from the end of the array. This also implicitly calls :ref:``delete`` on the removed element.

.. warning::
    If you use ``.length--`` on an empty array, it causes an underflow and thus sets the length to ``2**256-1``.

.. note::
    Increasing the length of a storage array has constant gas costs because storage is assumed to be zero-initialised, while decreasing the length has at least linear cost (but in most cases worse than linear), because it includes explicitly clearing the removed elements similar to calling :ref:``delete`` on them.

.. note::
    It is not yet possible to use arrays of arrays in external functions (but they are supported in public functions).

.. note::
    In EVM versions before Byzantium, it was not possible to access dynamic arrays return from function calls. If you call functions that return dynamic arrays, make sure to use an EVM that is set to
    Byzantium mode.

::

    pragma solidity >=0.4.16 <0.6.0;

    contract ArrayContract {
        uint[2**20] m_aLotOfIntegers;
        // Note that the following is not a pair of dynamic arrays but a
        // dynamic array of pairs (i.e. of fixed size arrays of length two).
        // Because of that, T[] is always a dynamic array of T, even if T
        // itself is an array.
        // Data location for all state variables is storage.
        bool[2][] m_pairsOfFlags;

        // newPairs is stored in memory - the only possibility
        // for public contract function arguments
        function setAllFlagPairs(bool[2][] memory newPairs) public {
            // assignment to a storage array performs a copy of ``newPairs`` and
            // replaces the complete array ``m_pairsOfFlags``.
            m_pairsOfFlags = newPairs;
        }

        struct StructType {
            uint[] contents;
            uint moreInfo;
        }
        StructType s;

        function f(uint[] memory c) public {
            // stores a reference to ``s`` in ``g``
            StructType storage g = s;
            // also changes ``s.moreInfo``.
            g.moreInfo = 2;
            // assigns a copy because ``g.contents``
            // is not a local variable, but a member of
            // a local variable.
            g.contents = c;
        }

        function setFlagPair(uint index, bool flagA, bool flagB) public {
            // access to a non-existing index will throw an exception
            m_pairsOfFlags[index][0] = flagA;
            m_pairsOfFlags[index][1] = flagB;
        }

        function changeFlagArraySize(uint newSize) public {
            // if the new size is smaller, removed array elements will be cleared
            m_pairsOfFlags.length = newSize;
        }

        function clear() public {
            // these clear the arrays completely
            delete m_pairsOfFlags;
            delete m_aLotOfIntegers;
            // identical effect here
            m_pairsOfFlags.length = 0;
        }

        bytes m_byteData;

        function byteArrays(bytes memory data) public {
            // byte arrays ("bytes") are different as they are stored without padding,
            // but can be treated identical to "uint8[]"
            m_byteData = data;
            m_byteData.length += 7;
            m_byteData[3] = 0x08;
            delete m_byteData[2];
        }

        function addFlag(bool[2] memory flag) public returns (uint) {
            return m_pairsOfFlags.push(flag);
        }

        function createMemoryArray(uint size) public pure returns (bytes memory) {
            // Dynamic memory arrays are created using `new`:
            uint[2][] memory arrayOfPairs = new uint[2][](size);

            // Inline arrays are always statically-sized and if you only
            // use literals, you have to provide at least one type.
            arrayOfPairs[0] = [uint(1), 2];

            // Create a dynamic byte array:
            bytes memory b = new bytes(200);
            for (uint i = 0; i < b.length; i++)
                b[i] = byte(uint8(i));
            return b;
        }
    }


.. index:: ! struct, ! type;struct

.. _structs:

Structs
-------

Solidity provides a way to define new types in the form of structs, which is shown in the following example:

::

    pragma solidity >=0.4.11 <0.6.0;

    contract CrowdFunding {
        // Defines a new type with two fields.
        struct Funder {
            address addr;
            uint amount;
        }

        struct Campaign {
            address payable beneficiary;
            uint fundingGoal;
            uint numFunders;
            uint amount;
            mapping (uint => Funder) funders;
        }

        uint numCampaigns;
        mapping (uint => Campaign) campaigns;

        function newCampaign(address payable beneficiary, uint goal) public returns (uint campaignID) {
            campaignID = numCampaigns++; // campaignID is return variable
            // Creates new struct in memory and copies it to storage.
            // We leave out the mapping type, because it is not valid in memory.
            // If structs are copied (even from storage to storage), mapping types
            // are always omitted, because they cannot be enumerated.
            campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0);
        }

        function contribute(uint campaignID) public payable {
            Campaign storage c = campaigns[campaignID];
            // Creates a new temporary memory struct, initialised with the given values
            // and copies it over to storage.
            // Note that you can also use Funder(msg.sender, msg.value) to initialise.
            c.funders[c.numFunders++] = Funder({addr: msg.sender, amount: msg.value});
            c.amount += msg.value;
        }

        function checkGoalReached(uint campaignID) public returns (bool reached) {
            Campaign storage c = campaigns[campaignID];
            if (c.amount < c.fundingGoal)
                return false;
            uint amount = c.amount;
            c.amount = 0;
            c.beneficiary.transfer(amount);
            return true;
        }
    }

The contract does not provide the full functionality of a crowdfunding contract, but it contains the basic concepts necessary to understand structs.
Struct types can be used inside mappings and arrays and they can itself contain mappings and arrays.

It is not possible for a struct to contain a member of its own type, although the struct itself can be the value type of a mapping member or it can contain a dynamically-sized array of its type.
This restriction is necessary, as the size of the struct has to be finite.

Note how in all the functions, a struct type is assigned to a local variable with data location ``storage``.
This does not copy the struct but only stores a reference so that assignments to members of the local variable actually write to the state.

Of course, you can also directly access the members of the struct without assigning it to a local variable, as in ``campaigns[campaignID].amount = 0``.

.. index:: !mapping

Mappings
--------

You declare mapping types with the syntax ``mapping(_KeyType => _ValueType)``.
The ``_KeyType`` can be any elementary type. This means it can be any of the built-in value types plus ``bytes`` and ``string``. User-defined or complex types like contract types, enums, mappings, structs and any array type apart from ``bytes`` and ``string`` are not allowed.
``_ValueType`` can be any type, including mappings.

You can think of mappings as `hash tables <https://en.wikipedia.org/wiki/Hash_table>`_, which are virtually initialised
such that every possible key exists and is mapped to a value whose byte-representation is all zeros, a type's :ref:`default value <default-value>`. The similarity ends there, the key data is not stored in a mapping, only its ``keccak256`` hash is used to look up the value.

Because of this, mappings do not have a length or a concept of a key or value being set.

Mappings can only have a data location of ``storage`` and thus are allowed for state variables, as storage reference types
in functions, or as parameters for library functions.
They cannot be used as parameters or return parameters of contract functions that are publicly visible.

You can mark variables of mapping type as ``public`` and Solidity creates a :ref:`getter <visibility-and-getters>` for you. The ``_KeyType`` becomes a parameter for the getter. If ``_ValueType`` is a value type or a struct, the getter returns ``_ValueType``.
If ``_ValueType`` is an array or a mapping, the getter has one parameter for each ``_KeyType``, recursively. For example with a mapping:

::

    pragma solidity >=0.4.0 <0.6.0;

    contract MappingExample {
        mapping(address => uint) public balances;

        function update(uint newBalance) public {
            balances[msg.sender] = newBalance;
        }
    }

    contract MappingUser {
        function f() public returns (uint) {
            MappingExample m = new MappingExample();
            m.update(100);
            return m.balances(address(this));
        }
    }


.. note::
  Mappings are not iterable, but it is possible to implement a data structure on top of them. For an example, see `iterable mapping <https://github.com/ethereum/dapp-bin/blob/master/library/iterable_mapping.sol>`_.

.. index:: assignment, ! delete, lvalue

Operators Involving LValues
===========================

If ``a`` is an LValue (i.e. a variable or something that can be assigned to), the following operators are available as shorthands:

``a += e`` is equivalent to ``a = a + e``. The operators ``-=``, ``*=``, ``/=``, ``%=``, ``|=``, ``&=`` and ``^=`` are defined accordingly. ``a++`` and ``a--`` are equivalent to ``a += 1`` / ``a -= 1`` but the expression itself still has the previous value of ``a``. In contrast, ``--a`` and ``++a`` have the same effect on ``a`` but return the value after the change.

delete
------

``delete a`` assigns the initial value for the type to ``a``. I.e. for integers it is equivalent to ``a = 0``, but it can also be used on arrays, where it assigns a dynamic array of length zero or a static array of the same length with all elements reset. For structs, it assigns a struct with all members reset. In other words, the value of ``a`` after ``delete a`` is the same as if ``a`` would be declared without assignment, with the following caveat:

``delete`` has no effect on mappings (as the keys of mappings may be arbitrary and are generally unknown). So if you delete a struct, it will reset all members that are not mappings and also recurse into the members unless they are mappings. However, individual keys and what they map to can be deleted: If ``a`` is a mapping, then ``delete a[x]`` will delete the value stored at ``x``.

It is important to note that ``delete a`` really behaves like an assignment to ``a``, i.e. it stores a new object in ``a``.
This distinction is visible when ``a`` is reference variable: It will only reset ``a`` itself, not the value it referred to previously.

::

    pragma solidity >=0.4.0 <0.6.0;

    contract DeleteExample {
        uint data;
        uint[] dataArray;

        function f() public {
            uint x = data;
            delete x; // sets x to 0, does not affect data
            delete data; // sets data to 0, does not affect x
            uint[] storage y = dataArray;
            delete dataArray; // this sets dataArray.length to zero, but as uint[] is a complex object, also
            // y is affected which is an alias to the storage object
            // On the other hand: "delete y" is not valid, as assignments to local variables
            // referencing storage objects can only be made from existing storage objects.
            assert(y.length == 0);
        }
    }

.. index:: ! type;conversion, ! cast

.. _types-conversion-elementary-types:

Conversions between Elementary Types
====================================

Implicit Conversions
--------------------

If an operator is applied to different types, the compiler tries to implicitly convert one of the operands to the type of the other (the same is true for assignments). In general, an implicit conversion between value-types is possible if it makes sense semantically and no information is lost: ``uint8`` is convertible to ``uint16`` and ``int128`` to ``int256``, but ``int8`` is not convertible to ``uint256`` (because ``uint256`` cannot hold e.g. ``-1``).
Any integer type that can be converted to ``uint160`` can also be converted to ``address``.

For more details, please consult the sections about the types themselves.

Explicit Conversions
--------------------

If the compiler does not allow implicit conversion but you know what you are doing, an explicit type conversion is sometimes possible. Note that this may give you some unexpected behaviour and allows you to bypass some security features of the compiler, so be sure to test that the result is what you want! Take the following example where you are converting a negative ``int8`` to a ``uint``:

::

    int8 y = -3;
    uint x = uint(y);

At the end of this code snippet, ``x`` will have the value ``0xfffff..fd`` (64 hex characters), which is -3 in the two's complement representation of 256 bits.

If an integer is explicitly converted to a smaller type, higher-order bits are cut off::

    uint32 a = 0x12345678;
    uint16 b = uint16(a); // b will be 0x5678 now

If an integer is explicitly converted to a larger type, it is padded on the left (i.e. at the higher order end).
The result of the conversion will compare equal to the original integer.

    uint16 a = 0x1234;
    uint32 b = uint32(a); // b will be 0x00001234 now
    assert(a == b);

Fixed-size bytes types behave differently during conversions. They can be thought of as sequences of individual bytes and converting to a smaller type will cut off the sequence::

    bytes2 a = 0x1234;
    bytes1 b = bytes1(a); // b will be 0x12

If a fixed-size bytes type is explicitly converted to a larger type, it is padded on the right. Accessing the byte at a fixed index will result in the same value before and after the conversion (if the index is still in range)::

    bytes2 a = 0x1234;
    bytes4 b = bytes4(a); // b will be 0x12340000
    assert(a[0] == b[0]);
    assert(a[1] == b[1]);

Since integers and fixed-size byte arrays behave differently when truncating or padding, explicit conversions between integers and fixed-size byte arrays are only allowed, if both have the same size. If you want to convert between integers and fixed-size byte arrays of different size, you have to use intermediate conversions that make the desired truncation and padding
rules explicit::

    bytes2 a = 0x1234;
    uint32 b = uint16(a); // b will be 0x00001234
    uint32 c = uint32(bytes4(a)); // c will be 0x12340000
    uint8 d = uint8(uint16(a)); // d will be 0x34
    uint8 e = uint8(bytes1(a)); // d will be 0x12

.. _types-conversion-literals:

Conversions between Literals and Elementary Types
=================================================

Integer Types
-------------

Decimal and hexadecimal number literals can be implicitly converted to any integer type that is large enough to represent it without truncation::

    uint8 a = 12; // fine
    uint32 b = 1234; // fine
    uint16 c = 0x123456; // fails, since it would have to truncate to 0x3456

Fixed-Size Byte Arrays
----------------------

Decimal number literals cannot be implicitly converted to fixed-size byte arrays. Hexadecimal number literals can be, but only if the number of hex digits exactly fits the size of the bytes type. As an exception both decimal and hexadecimal literals which have a value of zero can be converted to any fixed-size bytes type::

    bytes2 a = 54321; // not allowed
    bytes2 b = 0x12; // not allowed
    bytes2 c = 0x123; // not allowed
    bytes2 d = 0x1234; // fine
    bytes2 e = 0x0012; // fine
    bytes4 f = 0; // fine
    bytes4 g = 0x0; // fine

String literals and hex string literals can be implicitly converted to fixed-size byte arrays, if their number of characters matches the size of the bytes type::

    bytes2 a = hex"1234"; // fine
    bytes2 b = "xy"; // fine
    bytes2 c = hex"12"; // not allowed
    bytes2 d = hex"123"; // not allowed
    bytes2 e = "x"; // not allowed
    bytes2 f = "xyz"; // not allowed

Addresses
---------

As described in :ref:`address_literals`, hex literals of the correct size that pass the checksum test are of ``address`` type. No other literals can be implicitly converted to the ``address`` type.

Explicit conversions from ``bytes20`` or any integer type to ``address`` results in ``address payable``.