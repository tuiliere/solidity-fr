.. index:: contract, state variable, function, event, struct, enum, function;modifier

.. _contract_structure:

**********************
Structure d'un Contrat
**********************

Les contrats dans Solidity sont similaires à des classes dans les langages orientés objet.
Chaque contrat peut contenir des déclarations de :ref:`structure-state-variables`, :ref:`structure-fonctions`, :ref:`structure-fonction-modificateurs`, :ref:`structure-events`, :ref:`structure-struct-types` et :ref:`structure-enum-types`.
De plus, les contrats peuvent hériter d'autres contrats.

Il existe également des types de contrats spéciaux appelés :ref:`librairies<libraries>` et :ref:`interfaces<interfaces>`.

La section sur les :ref:`contrats<contracts>` contient plus de détails que cette section, qui sert à donner un bref aperçu.

.. _structure-state-variables:

Variables d'État
================

Les variables d'état sont des variables dont les valeurs sont stockées en permanence dans la mémoire du contrat.

::

    pragma solidity >=0.4.0 <0.6.0;

    contract SimpleStorage {
        uint storedData; // State variable
        // ...
    }

Voir la section :ref:`types` pour les types de variables d'état valides et :ref:`visibility-and-getters` pour les choix possibles de visibilité.

.. _structure-functions:

Fonctions
=========

Les fonctions sont les unités exécutables du code d'un contrat.

::

    pragma solidity >=0.4.0 <0.6.0;

    contract SimpleAuction {
        function bid() public payable { // Fonction
            // ...
        }
    }

Les :ref:`function-calls` peuvent se produire en interne ou en externe et avoir différents niveaux de :ref:`visibilité<visibility-and-getters>` vers d'autres contrats.

.. _structure-function-modifiers:

Function Modifiers
==================

Function modifiers can be used to amend the semantics of functions in a declarative way (see :ref:`modifiers` in the contracts section).

::

    pragma solidity >=0.4.22 <0.6.0;

    contract Purchase {
        address public seller;

        modifier onlySeller() { // Modifier
            require(
                msg.sender == seller,
                "Only seller can call this."
            );
            _;
        }

        function abort() public view onlySeller { // Modifier usage
            // ...
        }
    }

.. _structure-events:

Events
======

Events are convenience interfaces with the EVM logging facilities.

::

    pragma solidity >=0.4.21 <0.6.0;

    contract SimpleAuction {
        event HighestBidIncreased(address bidder, uint amount); // Event

        function bid() public payable {
            // ...
            emit HighestBidIncreased(msg.sender, msg.value); // Triggering event
        }
    }

See :ref:`events` in contracts section for information on how events are declared and can be used from within a dapp.

.. _structure-struct-types:

Struct Types
=============

Structs are custom defined types that can group several variables (see :ref:`structs` in types section).

::

    pragma solidity >=0.4.0 <0.6.0;

    contract Ballot {
        struct Voter { // Struct
            uint weight;
            bool voted;
            address delegate;
            uint vote;
        }
    }

.. _structure-enum-types:

Enum Types
==========

Enums can be used to create custom types with a finite set of 'constant values' (see :ref:`enums` in types section).

::

    pragma solidity >=0.4.0 <0.6.0;

    contract Purchase {
        enum State { Created, Locked, Inactive } // Enum
    }
