###################
Assembleur Solidity
###################

.. index:: ! assembly, ! asm, ! evmasm

Solidity définit un langage assembleur que vous pouvez utiliser sans Solidity et aussi comme l'assembleur en ligne ("inline assembly") dans le code source de Solidity. Ce guide commence par décrire comment utiliser l'assembleur en ligne, en quoi il diffère de l'assemblage autonome, et spécifie l'assemblage lui-même.

.. _inline-assembly:

Assembleur en ligne (inline)
============================

Vous pouvez entrelacer les instructions en Solidity avec de l'assembleur en ligne, dans un langage proche de celui de la machine virtuelle. Cela vous donne un contrôle plus fin, en particulier lorsque vous améliorez le langage en écrivant des bibliothèques.

Comme l'EVM est une machine à pile, il est souvent difficile d'adresser le bon emplacement de pile et de fournir des arguments aux opcodes au bon endroit sur la pile. L'assembleur en ligne de Solidity vous aide à le faire, ainsi qu'avec d'autres problèmes qui surviennent lors de la rédaction manuelle d'assembleur.

L'assembleur en ligne présente les caractéristiques suivantes :

* opcodes de type fonctionnels: ``mul(1, add(2, 3))``
* variables assembleur locales: ``let x := add(2, 3)  let y := mload(0x40)  x := add(x, y)``
* accèss à des variables externes: ``function f(uint x) public { assembly { x := sub(x, 1) } }``
* boucles: ``for { let i := 0 } lt(i, x) { i := add(i, 1) } { y := mul(2, y) }``
* conditions if: ``if slt(x, 0) { x := sub(0, x) }``
* conditions switch: ``switch x case 0 { y := mul(x, 2) } default { y := 0 }``
* appels de fonction: ``function f(x) -> y { switch x case 0 { y := 1 } default { y := mul(x, f(sub(x, 1))) }   }``

.. warning::
    L'assembleur en ligne est un moyen d'accéder à la machine virtuelle Ethereum en bas niveau. Ceci permet de contourner plusieurs normes de sécurité importantes et contrôles de Solidity. Vous ne devriez l'utiliser que pour les tâches qui en ont besoin, et seulement si vous êtes sûr de pourquoi/comment l'utiliser.

Syntaxe
-------

L'assembleur analyse les commentaires, les littéraux et les identificateurs de la même manière que Solidity, vous pouvez donc utiliser les commentaires habituels ``//`` et ``/* */``. L'assembleur en ligne est délimité par ``assembly { ... }`` et à l'intérieur de ces accolades, vous pouvez utiliser ce qui suit (voir les sections suivantes pour plus de détails) :

 - litéraux, par ex. ``0x123``, ``42`` ou ``"abc"`` (strings jusqu'à 32 caractères)
 - opcodes de type fonctionnels, exemple ``add(1, mlod(0))``
 - déclaration de variables, ex. ``let x := 7``, ``let x := add(y, 3)`` or ``let x`` (Initialisées à 0)
 - identifieurs (variables assembleur locales et externes si utilisée en tant qu'assembleur en ligne), ex. ``add(3, x)``, ``sstore(x_slot, 2)``
 - assignations, ex. ``x := add(y, 3)``
 - blocks de délimitation de portée, ex. ``{ let x := 3 { let y := add(x, 1) } }``

Les caractéristiques suivantes ne sont disponibles que dans l'assembleur utilisé seul:

 - contrôle direct de la stack ``dup1``, ``swap1``, ...
 - assignations directement sur la stack (de style instruction), e.g. ``3 =: x``
 - étiquettes, ex. ``name:``
 - opcodes de saut

.. note::
  L'assembleur autonome est rétrocompatible mais n'est plus documenté ici.

À la fin du bloc ``assembly { ... }``, la pile doit être équilibrée, à moins que vous n'en ayez besoin autrement. S'il n'est pas équilibré, le compilateur génère un avertissement.

Exemple
-------

L'exemple suivant fournit le code de bibliothèque pour accéder au code d'un autre contrat et le charger dans une variable ``bytes``. Ce n'est pas possible de base avec Solidity et l'idée est que les bibliothèques assembleur seront utilisées pour améliorer le langage Solidity.

.. code::

    pragma solidity >=0.4.0 <0.6.0;

    library GetCode {
        function at(address _addr) public view returns (bytes memory o_code) {
            assembly {
                // récupère la taille du code, a besoin d'assembleur
                let size := extcodesize(_addr)
                // allouer le tableau de bytes de sortie - ceci serait fait en Solidity via o_code = new bytes(size)
                o_code := mload(0x40)
                // nouvelle "fin de mémoire" en incluant le padding
                mstore(0x40, add(o_code, and(add(add(size, 0x20), 0x1f), not(0x1f))))
                // stocke la taille en mémoire
                mstore(o_code, size)
                // récupère le code lui-même, nécessite de l'assembleur
                extcodecopy(_addr, add(o_code, 0x20), 0, size)
            }
        }
    }

L'assembleur en ligne est également utile dans les cas où l'optimiseur ne parvient pas à produire un code efficace, par exemple :

.. code::

    pragma solidity >=0.4.16 <0.6.0;

    library VectorSum {
        // Cette fonction est moins efficace car l'optimiseur ne parvient
        // pas à supprimer les contrôles de limites dans l'accès aux tableaux.
        function sumSolidity(uint[] memory _data) public pure returns (uint o_sum) {
            for (uint i = 0; i < _data.length; ++i)
                o_sum += _data[i];
        }

        // Nous savons que nous n'accédons au tableau que dans ses
        // limites, ce qui nous permet d'éviter la vérification. 0x20
        // doit être ajouté à un tableau car le premier emplacement
        // contient la longueur du tableau.
        function sumAsm(uint[] memory _data) public pure returns (uint o_sum) {
            for (uint i = 0; i < _data.length; ++i) {
                assembly {
                    o_sum := add(o_sum, mload(add(add(_data, 0x20), mul(i, 0x20))))
                }
            }
        }

        // Même chose que ci-dessus, mais exécute le code entier en assembleur en ligne.
        function sumPureAsm(uint[] memory _data) public pure returns (uint o_sum) {
            assembly {
               // Charge la taille (premiers 32 bytes)
               let len := mload(_data)

               // Saute le champ de taille.
               //
               // Garde une variable temporaire pour pouvoir l'incrémenter.
               //
               // NOTE: incrémenter _data resulterait en une
               // variable _data inutilisable après ce bloc d'assembleur
               let data := add(_data, 0x20)

               // Itère jusqu'à la limite.
               for
                   { let end := add(data, mul(len, 0x20)) }
                   lt(data, end)
                   { data := add(data, 0x20) }
               {
                   o_sum := add(o_sum, mload(data))
               }
            }
        }
    }


.. _opcodes:

Opcodes
-------

Ce document ne se veut pas une description complète de la machine virtuelle Ethereum, mais la liste suivante peut être utilisée comme référence de ses opcodes.

Si un opcode prend des arguments (toujours du haut de la pile), ils sont donnés entre parenthèses.
Notez que l'ordre des arguments peut être vu comme étant inversé dans un style non fonctionnel (expliqué ci-dessous).
Les opcodes marqués par ``-`` ne poussent pas un article sur la pile, ceux marqués par ``*`` sont spéciaux et tous les autres poussent exactement une valeur sur la pile.
Les opcodes marqués avec ``F``, ``H``, ``B`` ou ``C`` sont présents depuis Frontier, Homestead, Byzantium ou Constantinople, respectivement.
Constantinople est toujours en cours de planification et toutes les instructions marquées comme telles entraîneront une exception d'instruction invalide à ce stade.

Dans ce qui suit, ``mem[a...b]`` signifie les octets de mémoire commençant à la position ``a`` jusqu'à la position ``b`` mais non comprise et ``storage[p]`` signifie le contenu du stockage à la position ``p``.

Les opcodes ``pushi`` et ``jumpdest`` ne peuvent pas être utilisés directement.

Dans la grammaire, les opcodes sont représentés comme des identificateurs prédéfinis.

+-------------------------+-----+---+-------------------------------------------------------------------+
| Instruction             |     |   | Explication                                                       |
+=========================+=====+===+===================================================================+
| stop                    + `-` | F | arrêt de l'exécution, identique à return(0,0)                     |
+-------------------------+-----+---+-------------------------------------------------------------------+
| add(x, y)               |     | F | x + y                                                             |
+-------------------------+-----+---+-------------------------------------------------------------------+
| sub(x, y)               |     | F | x - y                                                             |
+-------------------------+-----+---+-------------------------------------------------------------------+
| mul(x, y)               |     | F | x * y                                                             |
+-------------------------+-----+---+-------------------------------------------------------------------+
| div(x, y)               |     | F | x / y                                                             |
+-------------------------+-----+---+-------------------------------------------------------------------+
| sdiv(x, y)              |     | F | x / y, pour les nombres signés en complément à deux               |
+-------------------------+-----+---+-------------------------------------------------------------------+
| mod(x, y)               |     | F | x % y                                                             |
+-------------------------+-----+---+-------------------------------------------------------------------+
| smod(x, y)              |     | F | x % y, pour les nombres signés en complément à deux               |
+-------------------------+-----+---+-------------------------------------------------------------------+
| exp(x, y)               |     | F | x exposant y                                                      |
+-------------------------+-----+---+-------------------------------------------------------------------+
| not(x)                  |     | F | ~x, chaque bit de x est inversé                                   |
+-------------------------+-----+---+-------------------------------------------------------------------+
| lt(x, y)                |     | F | 1 si x < y, 0 sinon                                               |
+-------------------------+-----+---+-------------------------------------------------------------------+
| gt(x, y)                |     | F | 1 si x > y, 0 sinon                                               |
+-------------------------+-----+---+-------------------------------------------------------------------+
| slt(x, y)               |     | F | 1 si x < y, 0 sinon, pour les nombres signés en complément à deux |
+-------------------------+-----+---+-------------------------------------------------------------------+
| sgt(x, y)               |     | F | 1 si x > y, 0 sinon, pour les nombres signés en complément à deux |
+-------------------------+-----+---+-------------------------------------------------------------------+
| eq(x, y)                |     | F | 1 si x == y, 0 sinon                                              |
+-------------------------+-----+---+-------------------------------------------------------------------+
| iszero(x)               |     | F | 1 si x == 0, 0 sinon                                              |
+-------------------------+-----+---+-------------------------------------------------------------------+
| and(x, y)               |     | F | and binaire de x et y                                             |
+-------------------------+-----+---+-------------------------------------------------------------------+
| or(x, y)                |     | F | or binaire de x et y                                              |
+-------------------------+-----+---+-------------------------------------------------------------------+
| xor(x, y)               |     | F | xor binaire de x et y                                             |
+-------------------------+-----+---+-------------------------------------------------------------------+
| byte(n, x)              |     | F | nème octet de x, où le bit de poids fort est le 0ème              |
+-------------------------+-----+---+-------------------------------------------------------------------+
| shl(x, y)               |     | C | décalage logique binaire de y à gauche de x bits                  |
+-------------------------+-----+---+-------------------------------------------------------------------+
| shr(x, y)               |     | C | décalage logique binaire de y à droite de x bits                  |
+-------------------------+-----+---+-------------------------------------------------------------------+
| sar(x, y)               |     | C | décalage arithmétique de y à droite de x bits                     |
+-------------------------+-----+---+-------------------------------------------------------------------+
| addmod(x, y, m)         |     | F | (x + y) % m arithmétique de précision arbitraire                  |
+-------------------------+-----+---+-------------------------------------------------------------------+
| mulmod(x, y, m)         |     | F | (x * y) % m arithmétique de précision arbitraire                  |
+-------------------------+-----+---+-------------------------------------------------------------------+
| signextend(i, x)        |     | F | signe déplacé au (i*8+7)ème bit en partant du bit de poids faible |
+-------------------------+-----+---+-------------------------------------------------------------------+
| keccak256(p, n)         |     | F | keccak(mem[p...(p+n)))                                            |
+-------------------------+-----+---+-------------------------------------------------------------------+
| jump(label)             | `-` | F | saute à l'étiquette / position dans le code                       |
+-------------------------+-----+---+-------------------------------------------------------------------+
| jumpi(label, cond)      | `-` | F | saute à l'étiquette si cond différent de 0                        |
+-------------------------+-----+---+-------------------------------------------------------------------+
| pc                      |     | F | position actuelle dans le code                                    |
+-------------------------+-----+---+-------------------------------------------------------------------+
| pop(x)                  | `-` | F | retire l'élément poussé sur la stack par x                        |
+-------------------------+-----+---+-------------------------------------------------------------------+
| dup1 ... dup16          |     | F | copie le nième emplacement (du haut) de la pile sur le dessus     |
+-------------------------+-----+---+-------------------------------------------------------------------+
| swap1 ... swap16        | `*` | F | échange l'élément du dessus de la pile avec le nième sous lui-même|
+-------------------------+-----+---+-------------------------------------------------------------------+
| mload(p)                |     | F | mem[p...(p+32))                                                   |
+-------------------------+-----+---+-------------------------------------------------------------------+
| mstore(p, v)            | `-` | F | mem[p...(p+32)) := v                                              |
+-------------------------+-----+---+-------------------------------------------------------------------+
| mstore8(p, v)           | `-` | F | mem[p] := v & 0xff (modifie uniquement un bit)                    |
+-------------------------+-----+---+-------------------------------------------------------------------+
| sload(p)                |     | F | storage[p]                                                        |
+-------------------------+-----+---+-------------------------------------------------------------------+
| sstore(p, v)            | `-` | F | storage[p] := v                                                   |
+-------------------------+-----+---+-------------------------------------------------------------------+
| msize                   |     | F | taille actuelle de memory, c.à.d plus grand index mémoire         |
+-------------------------+-----+---+-------------------------------------------------------------------+
| gas                     |     | F | gas toujours disponible à l'éxécution                             |
+-------------------------+-----+---+-------------------------------------------------------------------+
| address                 |     | F | addresse du contrat en cours / du contexte d'éxécution            |
+-------------------------+-----+---+-------------------------------------------------------------------+
| balance(a)              |     | F | solde en wei de l'adresse a                                       |
+-------------------------+-----+---+-------------------------------------------------------------------+
| caller                  |     | F | emetteur du message (excluant ``delegatecall``)                   |
+-------------------------+-----+---+-------------------------------------------------------------------+
| callvalue               |     | F | wei envoyés avec l'appel courant                                  |
+-------------------------+-----+---+-------------------------------------------------------------------+
| calldataload(p)         |     | F | données d'appel (call data) commençant à la position p (32 octets)|
+-------------------------+-----+---+-------------------------------------------------------------------+
| calldatasize            |     | F | taille des données d'appel en octets                              |
+-------------------------+-----+---+-------------------------------------------------------------------+
| calldatacopy(t, f, s)   | `-` | F | copie s octets de la position f de calldata vers t en memoire     |
+-------------------------+-----+---+-------------------------------------------------------------------+
| codesize                |     | F | size of the code of the current contract / execution context      |
+-------------------------+-----+---+-------------------------------------------------------------------+
| codecopy(t, f, s)       | `-` | F | copy s bytes from code at position f to mem at position t         |
+-------------------------+-----+---+-------------------------------------------------------------------+
| extcodesize(a)          |     | F | size of the code at address a                                     |
+-------------------------+-----+---+-------------------------------------------------------------------+
| extcodecopy(a, t, f, s) | `-` | F | comme codecopy(t, f, s) mais prend le code à l'adresse a          |
+-------------------------+-----+---+-------------------------------------------------------------------+
| returndatasize          |     | B | taille du dernier returndata                                      |
+-------------------------+-----+---+-------------------------------------------------------------------+
| returndatacopy(t, f, s) | `-` | B | copie s octets de la position f de returndata vers t en mémoire   |
+-------------------------+-----+---+-------------------------------------------------------------------+
| extcodehash(a)          |     | C | hash du code de l'adresse a                                       |
+-------------------------+-----+---+-------------------------------------------------------------------+
| create(v, p, n)         |     | F | créée un nouveau contrat avec le code mem[p...(p+n)) et envoie v  |
|                         |     |   | wei puis retourne la nouvelle adresse                             |
+-------------------------+-----+---+-------------------------------------------------------------------+
| create2(v, p, n, s)     |     | C | créée un nouveau contrat avec le code mem[p...(p+n)) à l'adresse  |
|                         |     |   | keccak256(0xff . this . s . keccak256(mem[p...(p+n)))             |
|                         |     |   | et envoie v wei puis retourne la nouvelle adresse, où ``0xff`` est|
|                         |     |   | une valeur sur 8 octets, ``this`` est l'adresse du contrat courant|
|                         |     |   | sur 20 octets et ``s`` est une valeur 256 bits en big-endian      |
+-------------------------+-----+---+-------------------------------------------------------------------+
| call(g, a, v, in,       |     | F | appelle le contrat à l'adresse a avec les données d'entrée        |
| insize, out, outsize)   |     |   | mem[in...(in+insize)), en fournissant g gas et v wei et l'espace  |
|                         |     |   | mémoire de sortie mem[out...(out+outsize)), retournant 0 en cas   |
|                         |     |   | d'erreur (ex. manque de gas) et 1 en cas de succès                |
+-------------------------+-----+---+-------------------------------------------------------------------+
| callcode(g, a, v, in,   |     | F | identique à ``call`` mais utilise seulement le code de a en       |
| insize, out, outsize)   |     |   | restant dans le contexte du contrat courant                       |
+-------------------------+-----+---+-------------------------------------------------------------------+
| delegatecall(g, a, in,  |     | H | identique à ``callcode`` mais garde également ``caller``          |
| insize, out, outsize)   |     |   | et ``callvalue``                                                  |
+-------------------------+-----+---+-------------------------------------------------------------------+
| staticcall(g, a, in,    |     | B | identique à ``call(g, a, 0, in, insize, out, outsize)`` mais      |
| insize, out, outsize)   |     |   | n'autorise pasd e modifications de l'état                         |
+-------------------------+-----+---+-------------------------------------------------------------------+
| return(p, s)            | `-` | F | termine l'éxécution, retourne data mem[p...(p+s))                 |
+-------------------------+-----+---+-------------------------------------------------------------------+
| revert(p, s)            | `-` | B | termine l'éxécution, annule les changement de l'état, retourne    |
|                         |     |   | data mem[p...(p+s))                                               |
+-------------------------+-----+---+-------------------------------------------------------------------+
| selfdestruct(a)         | `-` | F | termine l'éxécution, destroy current contract and send funds to a |
+-------------------------+-----+---+-------------------------------------------------------------------+
| invalid                 | `-` | F | termine l'éxécution with invalid instruction                      |
+-------------------------+-----+---+-------------------------------------------------------------------+
| log0(p, s)              | `-` | F | ajoute data mem[p...(p+s)) au journal sans topics                 |
+-------------------------+-----+---+-------------------------------------------------------------------+
| log1(p, s, t1)          | `-` | F | ajoute data mem[p...(p+s)) au journal avec le topic t1            |
+-------------------------+-----+---+-------------------------------------------------------------------+
| log2(p, s, t1, t2)      | `-` | F | ajoute data mem[p...(p+s)) au journal avec les topics t1 et t2    |
+-------------------------+-----+---+-------------------------------------------------------------------+
| log3(p, s, t1, t2, t3)  | `-` | F | ajoute data mem[p...(p+s)) au journal avec les topics t1, t2 et t3|
+-------------------------+-----+---+-------------------------------------------------------------------+
| log4(p, s, t1, t2, t3,  | `-` | F | ajoute data mem[p...(p+s)) au journal avec topics t1, t2, t3 et t4|
| t4)                     |     |   |                                                                   |
+-------------------------+-----+---+-------------------------------------------------------------------+
| origin                  |     | F | emetteur de la transaction                                        |
+-------------------------+-----+---+-------------------------------------------------------------------+
| gasprice                |     | F | prix du gas pour cette transaction                                |
+-------------------------+-----+---+-------------------------------------------------------------------+
| blockhash(b)            |     | F | hash du bloc numero b                                             |
|                         |     |   | seulement pour els derniers 256 blocs excluant le courant         |
+-------------------------+-----+---+-------------------------------------------------------------------+
| coinbase                |     | F | bénéficiaire du minage courant                                    |
+-------------------------+-----+---+-------------------------------------------------------------------+
| timestamp               |     | F | timestamp du bloc courant en secondes depuis l'epoch UNIX         |
+-------------------------+-----+---+-------------------------------------------------------------------+
| number                  |     | F | numéro du bloc courant                                            |
+-------------------------+-----+---+-------------------------------------------------------------------+
| difficulty              |     | F | difficulté du bloc courant                                        |
+-------------------------+-----+---+-------------------------------------------------------------------+
| gaslimit                |     | F | limite de gas du bloc courant                                     |
+-------------------------+-----+---+-------------------------------------------------------------------+

Literaux
--------

You can use integer constants by typing them in decimal or hexadecimal notation and an appropriate ``PUSHi`` instruction will automatically be generated. The following creates code to add 2 and 3 resulting in 5 and then computes the bitwise and with the string "abc".
The final value is assigned to a local variable called ``x``.
Strings are stored left-aligned and cannot be longer than 32 bytes.

.. code::

    assembly { let x := and("abc", add(3, 2)) }


Functional Style
-----------------

For a sequence of opcodes, it is often hard to see what the actual arguments for certain opcodes are. In the following example, ``3`` is added to the contents in memory at position ``0x80``.

.. code::

    3 0x80 mload add 0x80 mstore

Solidity inline assembly has a "functional style" notation where the same code would be written as follows:

.. code::

    mstore(0x80, add(mload(0x80), 3))

If you read the code from right to left, you end up with exactly the same sequence of constants and opcodes, but it is much clearer where the values end up.

If you care about the exact stack layout, just note that the syntactically first argument for a function or opcode will be put at the top of the stack.

Access to External Variables, Functions and Libraries
-----------------------------------------------------

You can access Solidity variables and other identifiers by using their name.
For variables stored in the memory data location, this pushes the address, and not the value onto the stack. Variables stored in the storage data location are different, as they might not occupy a full storage slot, so their "address" is composed of a slot and a byte-offset inside that slot. To retrieve the slot pointed to by the variable ``x``, you use ``x_slot``, and to retrieve the byte-offset you use ``x_offset``.

Local Solidity variables are available for assignments, for example:

.. code::

    pragma solidity >=0.4.11 <0.6.0;

    contract C {
        uint b;
        function f(uint x) public view returns (uint r) {
            assembly {
                r := mul(x, sload(b_slot)) // ignore the offset, we know it is zero
            }
        }
    }

.. warning::
    If you access variables of a type that spans less than 256 bits (for example ``uint64``, ``address``, ``bytes16`` or ``byte``), you cannot make any assumptions about bits not part of the encoding of the type. Especially, do not assume them to be zero.
    To be safe, always clear the data properly before you use it in a context where this is important:
    ``uint32 x = f(); assembly { x := and(x, 0xffffffff) /* now use x */ }``
    To clean signed types, you can use the ``signextend`` opcode.

Labels
------

Support for labels has been removed in version 0.5.0 of Solidity.
Please use functions, loops, if or switch statements instead.

Declaring Assembly-Local Variables
----------------------------------

You can use the ``let`` keyword to declare variables that are only visible in inline assembly and actually only in the current ``{...}``-block. What happens is that the ``let`` instruction will create a new stack slot that is reserved
for the variable and automatically removed again when the end of the block is reached. You need to provide an initial value for the variable which can be just ``0``, but it can also be a complex functional-style expression.

.. code::

    pragma solidity >=0.4.16 <0.6.0;

    contract C {
        function f(uint x) public view returns (uint b) {
            assembly {
                let v := add(x, 1)
                mstore(0x80, v)
                {
                    let y := add(sload(v), 1)
                    b := y
                } // y is "deallocated" here
                b := add(b, v)
            } // v is "deallocated" here
        }
    }


Assignments
-----------

Assignments are possible to assembly-local variables and to function-local variables. Take care that when you assign to variables that point to memory or storage, you will only change the pointer and not the data.

Variables can only be assigned expressions that result in exactly one value.
If you want to assign the values returned from a function that has multiple return parameters, you have to provide multiple variables.

.. code::

    {
        let v := 0
        let g := add(v, 2)
        function f() -> a, b { }
        let c, d := f()
    }

If
--

The if statement can be used for conditionally executing code.
There is no "else" part, consider using "switch" (see below) if you need multiple alternatives.

.. code::

    {
        if eq(value, 0) { revert(0, 0) }
    }

The curly braces for the body are required.

Switch
------

You can use a switch statement as a very basic version of "if/else".
It takes the value of an expression and compares it to several constants.
The branch corresponding to the matching constant is taken. Contrary to the error-prone behaviour of some programming languages, control flow does not continue from one case to the next. There can be a fallback or default case called ``default``.

.. code::

    {
        let x := 0
        switch calldataload(4)
        case 0 {
            x := calldataload(0x24)
        }
        default {
            x := calldataload(0x44)
        }
        sstore(0, div(x, 2))
    }

The list of cases does not require curly braces, but the body of a case does require them.

Loops
-----

Assembly supports a simple for-style loop. For-style loops have a header containing an initializing part, a condition and a post-iteration part. The condition has to be a functional-style expression, while the other two are blocks. If the initializing part declares any variables, the scope of these variables is extended into the body (including the condition and the post-iteration part).

The following example computes the sum of an area in memory.

.. code::

    {
        let x := 0
        for { let i := 0 } lt(i, 0x100) { i := add(i, 0x20) } {
            x := add(x, mload(i))
        }
    }

For loops can also be written so that they behave like while loops:
Simply leave the initialization and post-iteration parts empty.

.. code::

    {
        let x := 0
        let i := 0
        for { } lt(i, 0x100) { } {     // while(i < 0x100)
            x := add(x, mload(i))
            i := add(i, 0x20)
        }
    }

Functions
---------

Assembly allows the definition of low-level functions. These take their arguments (and a return PC) from the stack and also put the results onto the stack. Calling a function looks the same way as executing a functional-style opcode.

Functions can be defined anywhere and are visible in the block they are declared in. Inside a function, you cannot access local variables defined outside of that function. There is no explicit ``return`` statement.

If you call a function that returns multiple values, you have to assign them to a tuple using ``a, b := f(x)`` or ``let a, b := f(x)``.

The following example implements the power function by square-and-multiply.

.. code::

    {
        function power(base, exponent) -> result {
            switch exponent
            case 0 { result := 1 }
            case 1 { result := base }
            default {
                result := power(mul(base, base), div(exponent, 2))
                switch mod(exponent, 2)
                    case 1 { result := mul(base, result) }
            }
        }
    }

Things to Avoid
---------------

Inline assembly might have a quite high-level look, but it actually is extremely low-level. Function calls, loops, ifs and switches are converted by simple rewriting rules and after that, the only thing the assembler does for you is re-arranging
functional-style opcodes, counting stack height for variable access and removing stack slots for assembly-local variables when the end of their block is reached.

Conventions in Solidity
-----------------------

In contrast to EVM assembly, Solidity knows types which are narrower than 256 bits, e.g. ``uint24``. In order to make them more efficient, most arithmetic operations just treat them as 256-bit numbers and the higher-order bits are only cleaned at the point where it is necessary, i.e. just shortly before they are written to memory or before comparisons are performed. This means that if you access such a variable from within inline assembly, you might have to manually clean the higher order bits first.

Solidity manages memory in a very simple way: There is a "free memory pointer" at position ``0x40`` in memory. If you want to allocate memory, just use the memory starting from where this pointer points at and update it accordingly.
There is no guarantee that the memory has not been used before and thus you cannot assume that its contents are zero bytes.
There is no built-in mechanism to release or free allocated memory.
Here is an assembly snippet that can be used for allocating memory::

    function allocate(length) -> pos {
      pos := mload(0x40)
      mstore(0x40, add(pos, length))
    }

The first 64 bytes of memory can be used as "scratch space" for short-term allocation. The 32 bytes after the free memory pointer (i.e. starting at ``0x60``) is meant to be zero permanently and is used as the initial value for empty dynamic memory arrays.
This means that the allocatable memory starts at ``0x80``, which is the initial value of the free memory pointer.

Elements in memory arrays in Solidity always occupy multiples of 32 bytes (yes, this is even true for ``byte[]``, but not for ``bytes`` and ``string``). Multi-dimensional memory arrays are pointers to memory arrays. The length of a dynamic array is stored at the first slot of the array and followed by the array elements.

.. warning::
    Statically-sized memory arrays do not have a length field, but it might be added later to allow better convertibility between statically- and dynamically-sized arrays, so please do not rely on that.


Standalone Assembly
===================

The assembly language described as inline assembly above can also be used standalone and in fact, the plan is to use it as an intermediate language for the Solidity compiler. In this form, it tries to achieve several goals:

1. Programs written in it should be readable, even if the code is generated by a compiler from Solidity.
2. The translation from assembly to bytecode should contain as few "surprises" as possible.
3. Control flow should be easy to detect to help in formal verification and optimization.

In order to achieve the first and last goal, assembly provides high-level constructs like ``for`` loops, ``if`` and ``switch`` statements and function calls. It should be possible to write assembly programs that do not make use of explicit ``SWAP``, ``DUP``, ``JUMP`` and ``JUMPI`` statements, because the first two obfuscate the data flow and the last two obfuscate control flow. Furthermore, functional statements of the form ``mul(add(x, y), 7)`` are preferred over pure opcode statements like
``7 y x add mul`` because in the first form, it is much easier to see which operand is used for which opcode.

The second goal is achieved by compiling the higher level constructs to bytecode in a very regular way.
The only non-local operation performed by the assembler is name lookup of user-defined identifiers (functions, variables, ...), which follow very simple and regular scoping rules and cleanup of local variables from the stack.

Scoping: An identifier that is declared (label, variable, function, assembly) is only visible in the block where it was declared (including nested blocks inside the current block). It is not legal to access local variables across function borders, even if they would be in scope. Shadowing is not allowed.
Local variables cannot be accessed before they were declared, but functions and assemblies can. Assemblies are special blocks that are used for e.g. returning runtime code or creating contracts. No identifier from an outer assembly is visible in a sub-assembly.

If control flow passes over the end of a block, pop instructions are inserted that match the number of local variables declared in that block.
Whenever a local variable is referenced, the code generator needs to know its current relative position in the stack and thus it needs to keep track of the current so-called stack height. Since all local variables are removed at the end of a block, the stack height before and after the block should be the same. If this is not the case, compilation fails.

Using ``switch``, ``for`` and functions, it should be possible to write complex code without using ``jump`` or ``jumpi`` manually. This makes it much easier to analyze the control flow, which allows for improved formal verification and optimization.

Furthermore, if manual jumps are allowed, computing the stack height is rather complicated.
The position of all local variables on the stack needs to be known, otherwise neither references to local variables nor removing local variables automatically from the stack at the end of a block will work properly.

Example:

We will follow an example compilation from Solidity to assembly.
We consider the runtime bytecode of the following Solidity program::

    pragma solidity >=0.4.16 <0.6.0;

    contract C {
      function f(uint x) public pure returns (uint y) {
        y = 1;
        for (uint i = 0; i < x; i++)
          y = 2 * y;
      }
    }

The following assembly will be generated::

    {
      mstore(0x40, 0x80) // store the "free memory pointer"
      // function dispatcher
      switch div(calldataload(0), exp(2, 226))
      case 0xb3de648b {
        let r := f(calldataload(4))
        let ret := $allocate(0x20)
        mstore(ret, r)
        return(ret, 0x20)
      }
      default { revert(0, 0) }
      // memory allocator
      function $allocate(size) -> pos {
        pos := mload(0x40)
        mstore(0x40, add(pos, size))
      }
      // the contract function
      function f(x) -> y {
        y := 1
        for { let i := 0 } lt(i, x) { i := add(i, 1) } {
          y := mul(2, y)
        }
      }
    }


Assembly Grammar
----------------

The tasks of the parser are the following:

- Turn the byte stream into a token stream, discarding C++-style comments (a special comment exists for source references, but we will not explain it here).
- Turn the token stream into an AST according to the grammar below
- Register identifiers with the block they are defined in (annotation to the AST node) and note from which point on, variables can be accessed.

The assembly lexer follows the one defined by Solidity itself.

Whitespace is used to delimit tokens and it consists of the characters Space, Tab and Linefeed. Comments are regular JavaScript/C++ comments and are interpreted in the same way as Whitespace.

Grammar::

    AssemblyBlock = '{' AssemblyItem* '}'
    AssemblyItem =
        Identifier |
        AssemblyBlock |
        AssemblyExpression |
        AssemblyLocalDefinition |
        AssemblyAssignment |
        AssemblyStackAssignment |
        LabelDefinition |
        AssemblyIf |
        AssemblySwitch |
        AssemblyFunctionDefinition |
        AssemblyFor |
        'break' |
        'continue' |
        SubAssembly
    AssemblyExpression = AssemblyCall | Identifier | AssemblyLiteral
    AssemblyLiteral = NumberLiteral | StringLiteral | HexLiteral
    Identifier = [a-zA-Z_$] [a-zA-Z_0-9]*
    AssemblyCall = Identifier '(' ( AssemblyExpression ( ',' AssemblyExpression )* )? ')'
    AssemblyLocalDefinition = 'let' IdentifierOrList ( ':=' AssemblyExpression )?
    AssemblyAssignment = IdentifierOrList ':=' AssemblyExpression
    IdentifierOrList = Identifier | '(' IdentifierList ')'
    IdentifierList = Identifier ( ',' Identifier)*
    AssemblyStackAssignment = '=:' Identifier
    LabelDefinition = Identifier ':'
    AssemblyIf = 'if' AssemblyExpression AssemblyBlock
    AssemblySwitch = 'switch' AssemblyExpression AssemblyCase*
        ( 'default' AssemblyBlock )?
    AssemblyCase = 'case' AssemblyExpression AssemblyBlock
    AssemblyFunctionDefinition = 'function' Identifier '(' IdentifierList? ')'
        ( '->' '(' IdentifierList ')' )? AssemblyBlock
    AssemblyFor = 'for' ( AssemblyBlock | AssemblyExpression )
        AssemblyExpression ( AssemblyBlock | AssemblyExpression ) AssemblyBlock
    SubAssembly = 'assembly' Identifier AssemblyBlock
    NumberLiteral = HexNumber | DecimalNumber
    HexLiteral = 'hex' ('"' ([0-9a-fA-F]{2})* '"' | '\'' ([0-9a-fA-F]{2})* '\'')
    StringLiteral = '"' ([^"\r\n\\] | '\\' .)* '"'
    HexNumber = '0x' [0-9a-fA-F]+
    DecimalNumber = [0-9]+
