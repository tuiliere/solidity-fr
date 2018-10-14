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

Exemple d'utilisation des fonctions de type ``internal``::

    pragma solidity >=0.4.16 <0.6.0;

    library ArrayUtils {
      // les fonctions internes peuvent être utilisées dams des fonctions
      // de librairies internes car elles partagent le même contexte
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

Exemple d' usage de fonction ``external``::

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
        // Ici on checke que la réponse vient d'une source de confiance
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
    Les fonctions lambda ou en in-line sont prévues mais pas encore prises en charge.

.. index:: ! type;reference, ! reference type, storage, memory, location, array, struct

Types Référence
===============

Les valeurs du type référence peuvent être modifiées par plusieurs noms différents.
Comparez ceci avec les catégories de valeurs où vous obtenez une copie indépendante chaque fois qu'une variable de valeur est utilisée. Pour cette raison, les types référence doivent être traités avec plus d'attention que les types de valeur. Actuellement, les types référence comprennent les structures, les tableaux et les mappages. Si vous utilisez un type référence, vous devez toujours indiquer explicitement la zone de données où le type est enregistré : (dont la durée de vie est limitée à un appel de fonction), ``storage`` (l'emplacement où les variables d'état sont stockées) ou ``calldata`` (emplacement de données spécial qui contient les arguments de fonction, disponible uniquement pour les paramètres d'appel de fonction externe).

Une affectation ou une conversion de type qui modifie l'emplacement des données entraîne toujours une opération de copie automatique, alors que les affectations à l'intérieur du même emplacement de données ne copient que dans certains cas selon le type de stockage.

.. _data-location:

Emplacement des données
-----------------------

Chaque type référence, c'est-à-dire *arrays* (tableaux) et *structs*, comporte une annotation supplémentaire, la ``localisation des données``, indiquant où elles sont stockées. Il y a trois emplacements de données :
``Memory``, ``Storage`` et ``Calldata``. Calldata n'est valable que pour les paramètres des fonctions de contrat externes et n'est nécessaire que pour ce type de paramètre. Calldata est une zone non modifiable, non persistante où les arguments de fonction sont stockés, et se comporte principalement comme memory.


.. note::
    Avant la version 0.5.0, l'emplacement des données pouvait être omis, et était par défaut à des emplacements différents selon le type de variable, le type de fonction, etc.

La localisation des données n'est sont pas seulement pertinente pour la persistance des données, mais aussi pour la sémantique des affectations :
Les affectations entre le stockage et la mémoire (ou à partir des données de la calldata) créent toujours une copie indépendante.
Les affectations de mémoire à mémoire ne créent que des références. Cela signifie que les modifications d'une variable mémoire sont également visibles dans toutes les autres variables mémoire qui se réfèrent aux mêmes données.
Les affectations du stockage à une variable de stockage local n'affectent également qu'une référence.
En revanche, toutes les autres affectations au stockage sont toujours copiées. Les affectations à des variables d'état ou à des membres de variables locales de type structure de stockage, même si la variable locale elle-même n'est qu'une référence, constituent des exemples dans ce cas.

::

    pragma solidity >=0.4.0 <0.6.0;

    contract C {
        uint[] x; // the data location of x is storage

        // Les données de memoryArray sont stockées en mémoire (memory)
        function f(uint[] memory memoryArray) public {
            x = memoryArray; // marche, copie le tableau en storage
            uint[] storage y = x; // marche, assigne un pointeur, y est en storage
            y[7]; // bon, retourne le 8e élément
            y.length = 2; // bon, modifie x via y
            delete x; // bon, efface l'array, modifie y
            // L'exemple suivant ne fonctionne pas, il implique de créer un
            // tableau anonyme en storage, mais storage est alloué "statiquement"
            // y = memoryArray;
            // Ceci ne marche pas non plus, car ça redéfinirait le
            // pointeur mais ne pointe sur rien
            // delete y;
            g(x); // appelle g, avec un pointeur sur x
            h(x); // appelle h et crée une copie indépendante en memory
        }

        function g(uint[] storage) internal pure {}
        function h(uint[] memory) public pure {}
    }

.. index:: ! array

.. _arrays:

Tableaux
--------

Les tableaux peuvent avoir une taille fixe à la compilation ou peuvent être dynamiques.
Il y a peu de restrictions concernant l'élément contenu, il peut aussi être un autre tableau, un mappage ou une structure. Les restrictions générales
s'appliquent, cependant, en ce sens que les mappages ne peuvent être utilisés que dans le storage et que les fonctions visibles au public nécessitent des paramètres qui sont des types reconnus par l'ABI.

Un tableau de taille fixe ``k`` et de type d'élément ``T`` est écrit ``T[k]``, un tableau de taille dynamique ``T[]``. Par exemple, un tableau de 5 tableaux dynamiques de ``uint`` est ``uint[][5]`` (notez que la notation est inversée par rapport à certains autres langages). Pour accéder au deuxième uint du troisième tableau dynamique, vous utilisez ``x[2][1]`` (les indexs commencent à zéro et
l'accès fonctionne dans le sens inverse de la déclaration, c'est-à-dire que ``x[2]`` supprime un niveau dans le type de déclaration à partir de la droite).

L'accès à un tableau après sa fin provoque un ``revert``. Si vous voulez ajouter de nouveaux éléments, vous devez utiliser ``.push()`` ou augmenter le membre ``.length`` (voir ci-dessous).

Les variables de type ``bytes`` et ``string`` sont des tableaux spéciaux. Un ``byte`` est semblable à un ``byte[]``, mais il est condensé en calldata et en mémoire. ``string`` est égal à ``bytes``, mais ne permet pas l'accès à la longueur ou à l'index.
Il faut donc généralement préférer les ``bytes`` aux ``bytes[]`` car c'est moins cher à l'usage.
En règle générale, utilisez ``bytes`` pour les données en octets bruts de longueur arbitraire et ``string`` pour les données de chaîne de caractères de longueur arbitraire (UTF-8). Si vous pouvez limiter la longueur à un certain nombre d'octets, utilisez toujours un des ``bytes1`` à ``bytes32``, car ils sont beaucoup moins chers également.

.. note::
    Si vous voulez accéder à la représentation en octets d'une chaîne de caractères ``s``, utilisez ``bytes(s).length`` / ``bytes(s)[7] ='x';``. Gardez à l'esprit que vous accédez aux octets de bas niveau de la représentation UTF-8, et non aux caractères individuels !

Il est possible de marquer les tableaux ``public`` et de demander à Solidity de créer un :ref:`getter <visibility-and-getters>`.
L'index numérique deviendra un paramètre obligatoire pour le getter.

.. index:: ! array;allocating, new

Allouer des tableaux en mémoire
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Vous pouvez utiliser le mot-clé ``new`` pour créer des tableaux dont la longueur dépend de la durée d'exécution en mémoire.
Contrairement aux tableaux de stockage, il n'est **pas** possible de redimensionner les tableaux de mémoire (par exemple en les assignant au membre ``.length``). Vous devez soit calculer la taille requise à l'avance, soit créer un nouveau tableau de mémoire et copier chaque élément.

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

Tableaux littéraux / Inline Arrays
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Les littéraux de tableau sont des tableaux qui sont écrits comme une expression et ne sont pas assignés à une variable tout de suite.

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

Le type d'un tableau littéral est un tableau mémoire de taille fixe dont le type de base est le type commun des éléments donnés. Le type de ``[1, 2, 3]`` est ``uint8[3] memory```, car le type de chacune de ces constantes est ``uint8``.
Pour cette raison, il est nécessaire de convertir le premier élément de l'exemple ci-dessus en "uint". Notez qu'actuellement, les tableaux de taille fixe ne peuvent pas être assignées à des tableaux de taille dynamique, c'est-à-dire que ce qui suit n'est pas possible :

::

    pragma solidity >=0.4.0 <0.6.0;

    // Ceci ne compile pas.
    contract C {
        function f() public {
            // La ligne suivant provoque une erreur car uint[3] memory
            // ne peut pas être convertit en uint[] memory.
            uint[] memory x = [uint(1), 3, 4];
        }
    }

Il est prévu de supprimer cette restriction à l'avenir, mais crée actuellement certaines complications en raison de la façon dont les tableaux sont transmis dans l'ABI.

.. index:: ! array;length, length, push, pop, !array;push, !array;pop

Membres
^^^^^^^

**length**:
    Les tableaux ont un membre ``length`` qui contient leur nombre d'éléments.
     La longueur des tableaux memory est fixe (mais dynamique, c'est-à-dire qu'elle peut dépendre des paramètres d'exécution) une fois qu'ils sont créés.
     Pour les tableaux de taille dynamique (disponible uniquement en storage), ce membre peut être assigné pour redimensionner le tableau.
     L'accès à des éléments en dehors de la longueur courante ne redimensionne pas automatiquement le tableau et provoque plutôt un échec d'assertion.
     L'augmentation de la longueur ajoute de nouveaux éléments initialisés zéro au tableau.
     Réduire la longueur permet d'effectuer une suppression (:ref:``suppression``) implicite sur chacun des éléments supprimés.
**push** :
      Les tableaux de stockage dynamique et les ``bytes`` (et non ``string``) ont une fonction membre appelée ``push`` que vous pouvez utiliser pour ajouter un élément à la fin du tableau. L'élément sera mis à zéro à l'initialisation. La fonction renvoie la nouvelle longueur.
**pop** :
      Les tableaux de stockage dynamique et les ``bytes`` (et non ``string``) ont une fonction membre appelée ``pop`` que vous pouvez utiliser pour supprimer un élément à la fin du tableau. Ceci appelle aussi implicitement :ref:``delete`` sur l'élément supprimé.

.. warning::
    Si vous utilisez ``.length--`` sur un tableau vide, cela provoque un débordement par le bas et fixe donc la longueur à ``2**256-1``.

.. note::
     L'augmentation de la longueur d'un tableau en storage a des coûts en gas constants parce qu'on suppose que le stockage est nul, alors que la diminution de la longueur a au moins un coût linéaire (mais dans la plupart des cas pire que linéaire), parce qu'elle inclut explicitement l'élimination des éléments supprimés comme si on appelait :ref:``delete``.

.. note::
     Il n'est pas encore possible d'utiliser les tableaux de tableaux dans les fonctions externes (mais ils sont supportés dans les fonctions publiques).

.. note::
     Dans les versions EVM antérieures à Byzantium, il n'était pas possible d'accéder au retour de tableaux dynamique à partir des appels de fonctions. Si vous appelez des fonctions qui retournent des tableaux dynamiques, assurez-vous d'utiliser un EVM qui est configuré en mode Byzantium.

::

    pragma solidity >=0.4.16 <0.6.0;

    contract ArrayContract {
        uint[2**20] m_aLotOfIntegers;
        // Notez que ce qui suit n'est pas une paire de tableaux dynamiques
        // mais un tableau tableau dynamique de paires (c'est-à-dire de
        // tableaux de taille fixe de longueur deux).
        // Pour cette raison, T[] est toujours un tableau dynamique
        // de T, même si T lui-même est un tableau.
        // L'emplacement des données pour toutes les variables d'état
        // est storage.
        bool[2][] m_pairsOfFlags;

        // newPairs est stocké en memory - seule possibilité
        // pour les arguments de fonction publique
        function setAllFlagPairs(bool[2][] memory newPairs) public {
            // l'assignation d' un tableau en storage implique la copie
            // de  ``newPairs`` et remplace l'array ``m_pairsOfFlags``.
            m_pairsOfFlags = newPairs;
        }

        struct StructType {
            uint[] contents;
            uint moreInfo;
        }
        StructType s;

        function f(uint[] memory c) public {
            // stocke un pointeur sur ``s`` dans ``g``
            StructType storage g = s;
            // change aussi ``s.moreInfo``.
            g.moreInfo = 2;
            // assigne une copie car ``g.contents`` n'est
            // pas une variable locale mais un membre
            // d'une variable locale
            g.contents = c;
        }

        function setFlagPair(uint index, bool flagA, bool flagB) public {
            // accès à un index inexistant, déclenche une exception
            m_pairsOfFlags[index][0] = flagA;
            m_pairsOfFlags[index][1] = flagB;
        }

        function changeFlagArraySize(uint newSize) public {
            // Si la nouvelle taille est plus petite, les
            // éléments en trop seront supprimés
            m_pairsOfFlags.length = newSize;
        }

        function clear() public {
            // supprime le tableau complet
            delete m_pairsOfFlags;
            delete m_aLotOfIntegers;
            // meme effet avec
            m_pairsOfFlags.length = 0;
        }

        bytes m_byteData;

        function byteArrays(bytes memory data) public {
            // le tableau de byte ("bytes") sont différents car stockés sans
            // padding mais peuvent être traités comme des ``uint8[]``
            m_byteData = data;
            m_byteData.length += 7;
            m_byteData[3] = 0x08;
            delete m_byteData[2];
        }

        function addFlag(bool[2] memory flag) public returns (uint) {
            return m_pairsOfFlags.push(flag);
        }

        function createMemoryArray(uint size) public pure returns (bytes memory) {
            // Un tableau dynamique est créé via `new`:
            uint[2][] memory arrayOfPairs = new uint[2][](size);

            // Les tableaux déclarés à la volée sont toujours de taille statique
            // et en cas d' utilisation de littéraux uniquement, au moins
            // un type doit être spécifié.
            arrayOfPairs[0] = [uint(1), 2];

            // Créée un tableau dynamique de bytes:
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

Solidity permet de définir de nouveaux types sous forme de structs, comme le montre l'exemple suivant :

::

    pragma solidity >=0.4.11 <0.6.0;

    contract CrowdFunding {
        // Définit un nouveau type avec 2 champs.
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
            // Crée une nouvelle structure en memory et la copie vers storage.
            // Nous omettons le type de mappage, car il n'est pas valide en mémoire.
            // Si les structures sont copiées (même d'un stockage à un autre), les // types de mappage sont toujours omis, car ils ne peuvent pas
            // être énumérés.
            campaigns[campaignID] = Campaign(beneficiary, goal, 0, 0);
        }

        function contribute(uint campaignID) public payable {
            Campaign storage c = campaigns[campaignID];
            // Créé une nouvelle struct temporaire en memory, initialisée
            // aux valeurs voulues, et copie-la en storage.
            // Notez que vous pouvez également utiliser (msg.sender, msg.value)
            // pour l'initialiser.
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

Le contrat ne fournit pas toutes les fonctionnalités d'un contrat de crowdfunding, mais il contient les concepts de base nécessaires pour comprendre les ``struct``.
Les types structs peuvent être utilisés à l'intérieur des ``mapping`` et des ``array`` et peuvent eux-mêmes contenir des mappages et des tableaux.

Il n'est pas possible pour une structure de contenir un membre de son propre type, bien que la structure elle-même puisse être le type de valeur d'un membre de mappage ou peut contenir un tableau de taille dynamique de son type.
Cette restriction est nécessaire, car la taille de la structure doit être finie.

Notez que dans toutes les fonctions, un type structure est affecté à une variable locale avec l'emplacement de données ``storage``.
Ceci ne copie pas la structure mais stocke seulement une référence pour que les affectations aux membres de la variable locale écrivent réellement dans l'état.

Bien sûr, vous pouvez aussi accéder directement aux membres de la structure sans l'affecter à une variable locale, comme dans ``campaigns[campaignID].amount = 0``.

.. index:: !mapping

Mappages
--------

Vous déclarez le objets de type ``mapping`` avec la syntaxe ``mapping(_KeyType => _ValueType)``.
``_KeyType`` peut être n'importe quel type élémentaire. Cela signifie qu'il peut s'agir de n'importe lequel des types de valeurs intégrés plus les octets et les chaînes de caractères. Les types définis par l'utilisateur ou les types complexes tels que les types de contrat, les énuménations, les mappages, les structs et tout type de tableau, à l'exception des ``bytes`` et des ``string`` qui ne sont pas autorisés.
``_ValueType`` peut être n'importe quel type, y compris les mappages.

Vous pouvez considérer les mappings comme des `tables de hashage <https://fr.wikipedia.org/wiki/Table_de_hachage>`_, qui sont virtuellement initialisées de telle sorte que chaque clé possible existe et est mappée à une valeur dont la représentation binaire est constituée de zéros, de type :ref:`valeur par défaut <default-value>`. La similitude s'arrête là, les données "clés" ne sont pas stockées dans un mappage, seul son hachage ``keccak256`` est utilisé pour rechercher la valeur.

Pour cette raison, les mappages n'ont pas de longueur ou de concept de clé ou de valeur définie.

Les mappages ne peuvent avoir qu'un emplacement de données en ``storage`` et sont donc autorisés pour les variables d'état, comme types référence en storage dans les fonctions ou comme paramètres pour les fonctions de librairies.
Ils ne peuvent pas être utilisés comme paramètres ou paramètres de retour de fonctions de contrat publiques.

Vous pouvez marquer les variables de type mapping comme ``public`` et Solidity crée un :ref:`getter <visibility-and-getters>` pour vous. Le ```_KeyType`` devient un paramètre pour le getter. Si ``_ValueType`` est un type de valeur ou une structure, le getter retourne ``_ValueType``.
Si ``_ValueType`` est un tableau ou un mappage, le getter a un paramètre pour chaque ``_KeyType``, de manière récursive. Par exemple :

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
  Les mappings ne sont pas itérables, mais il est possible d'y ajouter une structure de données. Pour un exemple, voir `mapping iterable <https://github.com/ethereum/dapp-bin/blob/master/library/iterable_mapping.sol>`_.

.. index:: assignment, ! delete, lvalue

Opérateurs impliquant des LValues
=================================

Si ``a`` est une LValue (c.-à-d. une variable ou quelque chose qui peut être assigné à), les opérateurs suivants sont disponibles en version raccourcie::

``a += e`` équivaut à ``a = a + e``. Les opérateurs ``-=``, ``*=``, ``/=``, ``%=``, ``|=``, ``&=`` et ``^=`` sont définis de la même manière. ``a++`` et ``a--`` sont équivalents à ``a += 1`` / ``a -= 1`` mais l'expression elle-même a toujours la valeur précédente ``a``. Par contraste, ``--a`` et ``++a`` changent également ``a`` de ``1`` , mais retournent la valeur après le changement.

delete
------

``delete a`` affecte la valeur initiale du type à ``a``. C'est-à-dire que pour les entiers, il est équivalent à ``a = 0``, mais il peut aussi être utilisé sur les tableaux, où il assigne un tableau dynamique de longueur zéro ou un tableau statique de la même longueur avec tous les éléments réinitialisés. Pour les structs, il assigne une structure avec tous les membres réinitialisés. En d'autres termes, la valeur de ``a`` après ``delete a`` est la même que si ``a`` était déclaré sans attribution, avec la réserve suivante :

``delete`` n'a aucun effet sur les mappages (car les clés des mappages peuvent être arbitraires et sont généralement inconnues). Ainsi, si vous supprimez une structure, elle réinitialisera tous les membres qui ne sont pas des ``mappings`` et se propagera récursivement dans les membres à moins qu'ils ne soient des mappings. Toutefois, il est possible de supprimer des clés individuelles et ce à quoi elles correspondent : Si ``a`` est un mappage, alors ``delete a[x]`` supprimera la valeur stockée à ``x``.

Il est important de noter que ``delete a`` se comporte vraiment comme une affectation à ``a``, c'est-à-dire qu'il stocke un nouvel objet dans ``a``.
Cette distinction est visible lorsque ``a`` est une variable par référence : Il ne réinitialisera que ``a`` lui-même, et non la valeur à laquelle il se référait précédemment.

::

    pragma solidity >=0.4.0 <0.6.0;

    contract DeleteExample {
        uint data;
        uint[] dataArray;

        function f() public {
            uint x = data;
            delete x; // met x à 0, n' affecte pas data
            delete data; // met data à 0, n'affecte pas x
            uint[] storage y = dataArray;
            delete dataArray; // ceci met dataArray.length à zéro, mais un uint[]
            // est un objet complexe, donc y est affecté est un alias
            // vers l' objet en storage.
            // D' un autre côté: "delete y" est invalid, car l' assignement à
            // une variable locale pointant vers un objet en storage n' est
            // autorisée que depuis un objet en storage.
            assert(y.length == 0);
        }
    }

.. index:: ! type;conversion, ! cast

.. _types-conversion-elementary-types:

Conversions entre les types élémentaires
========================================

Conversions implicites
----------------------

Si un opérateur est appliqué à différents types, le compilateur essaie de convertir implicitement l'un des opérandes au type de l'autre (c'est la même chose pour les assignations). En général, une conversion implicite entre les types valeur est possible si elle a un sens sémantique et qu'aucune information n'est perdue : ``uint8`` est convertible en ``uint16`` et ``int128`` en ``int256``, mais ``uint8`` n'est pas convertible en ``uint256`` (car ``uint256`` ne peut contenir, par exemple, ``-1``).
Tout type d'entier qui peut être converti en ``uint160`` peut aussi être converti en ``address``.

Pour plus de détails, veuillez consulter les sections concernant les types eux-mêmes.

Conversions explicites
----------------------

Si le compilateur ne permet pas la conversion implicite mais que vous savez ce que vous faites, une conversion de type explicite est parfois possible. Notez que cela peut vous donner un comportement inattendu et vous permet de contourner certaines fonctions de sécurité du compilateur, donc assurez-vous de tester que le résultat est ce que vous voulez ! Prenons l'exemple suivant où l'on convertit un ``int8`` négatif en un ``uint`` :

::

    int8 y = -3;
    uint x = uint(y);

A la fin de cet extrait de code, ``x`` aura la valeur ``0xfffffff...fd`` (64 caractères hexadécimaux), qui est -3 dans la représentation en 256 bits du complément à deux.

Si un entier est explicitement converti en un type plus petit, les bits d'ordre supérieur sont coupés::

    uint32 a = 0x12345678;
    uint16 b = uint16(a); // b sera désormais 0x5678

Si un entier est explicitement converti en un type plus grand, il est rembourré par la gauche (c'est-à-dire à l'extrémité supérieure de l'ordre).
Le résultat de la conversion sera comparé à l'entier original::

    uint16 a = 0x1234;
    uint32 b = uint32(a); // b will be 0x00001234 now
    assert(a == b);

Les types à taille fixe se comportent différemment lors des conversions. Ils peuvent être considérés comme des séquences d'octets individuels et la conversion à un type plus petit coupera la séquence::

    bytes2 a = 0x1234;
    bytes1 b = bytes1(a); // b sera désormais 0x12

Si un type à taille fixe est explicitement converti en un type plus grand, il est rembourré à droite. L'accès à l'octet par un index fixe donnera la même valeur avant et après la conversion (si l'index est toujours dans la plage)::

    bytes2 a = 0x1234;
    bytes4 b = bytes4(a); // b sera désormais 0x12340000
    assert(a[0] == b[0]);
    assert(a[1] == b[1]);

Puisque les entiers et les tableaux d'octets de taille fixe se comportent différemment lorsqu'ils sont tronqués ou rembourrés, les conversions explicites entre entiers et tableaux d'octets de taille fixe ne sont autorisées que si les deux ont la même taille. Si vous voulez convertir entre des entiers et des tableaux d'octets de taille fixe de tailles différentes, vous devez utiliser des conversions intermédiaires qui font la troncature et le remplissage désirés.
règles explicites::

    bytes2 a = 0x1234;
    uint32 b = uint16(a); // b sera désormais 0x00001234
    uint32 c = uint32(bytes4(a)); // c sera désormais 0x12340000
    uint8 d = uint8(uint16(a)); // d sera désormais 0x34
    uint8 e = uint8(bytes1(a)); // d sera désormais 0x12

.. _types-conversion-literals:

Conversions entre les types littéraux et élémentaires
=====================================================

Types nombres entiers
-------------------

Les nombres décimaux et hexadécimaux peuvent être implicitement convertis en n'importe quel type entier suffisamment grand pour le représenter sans troncature::

    uint8 a = 12; // Bon
    uint32 b = 1234; // Bon
    uint16 c = 0x123456; // échoue, car devrait tronquer en 0x3456

Tableaux d'octets de taille fixe
--------------------------------

Les nombres décimaux ne peuvent pas être implicitement convertis en tableaux d'octets de taille fixe. Les nombres hexadécimaux peuvent être littéraux, mais seulement si le nombre de chiffres hexadécimaux correspond exactement à la taille du type de ``bytes``. Par exception, les nombres décimaux et hexadécimaux ayant une valeur de zéro peuvent être convertis en n'importe quel type à taille fixe::

    bytes2 a = 54321; // pas autorisé
    bytes2 b = 0x12; // pas autorisé
    bytes2 c = 0x123; // pas autorisé
    bytes2 d = 0x1234; // bon
    bytes2 e = 0x0012; // bon
    bytes4 f = 0; // bon
    bytes4 g = 0x0; // bon

Les littéraux de chaînes de caractères et les littéraux de chaînes hexadécimales peuvent être implicitement convertis en tableaux d'octets de taille fixe, si leur nombre de caractères correspond à la taille du type ``bytes``::

    bytes2 a = hex"1234"; // bon
    bytes2 b = "xy"; // bon
    bytes2 c = hex"12"; // pas autorisé
    bytes2 d = hex"123"; // pas autorisé
    bytes2 e = "x"; // pas autorisé
    bytes2 f = "xyz"; // débile

Adresses
--------

Comme décrit dans :ref:`address_literals`, les chaines de caractères hexadécimaux de la bonne taille qui passent le test de somme de contrôle sont de type ``address``. Aucun autre littéral ne peut être implicitement converti au type ``address``.

Les conversions explicites de ``bytes20`` ou de tout type entier en ``address`` aboutissent en une ``address payable```.