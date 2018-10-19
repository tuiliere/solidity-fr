.. index:: contract, state variable, function, event, struct, enum, function;modifier

.. _contract_structure:

**********************
Structure d'un contrat
**********************

Les contrats Solidity sont similaires à des classes dans des langages orientés objet.
Chaque contrat peut contenir des déclarations de :ref:`structure-state-variables`, :ref:`structure-fonctions`, :ref:`structure-fonction-modificateurs`, :ref:`structure-événements`, :ref:`structure-struct-types` et :ref:`structure-enum-types`.
De plus, les contrats peuvent hériter d'autres contrats.

Il existe également des types de contrats spéciaux appelés :ref:`libraries<libraries>` et :ref:`interfaces<interfaces>`.

La section sur les :ref:`contrats<contracts<contracts>` contient plus de détails que cette section, qui permet d'avoir une vue d'ensemble rapide.

.. _structure-state-variables :

Variables d'état
================

Les variables d'état sont des variables dont les valeurs sont stockées en permanence dans le storage du contrat.

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
        function bid() public payable { // Function
            // ...
        }
    }

Les :ref:`function-calls' peuvent se faire en interne ou en externe et ont différents niveaux de :ref:`visibilité<visibility-and-getters>` pour d'autres contrats.

.. _structure-function-modifiers:

Modificateurs de fonction
=========================

Les modificateurs de fonction peuvent être utilisés pour modifier la sémantique des fonctions d'une manière déclarative (voir :ref:`modifiers` dans la section contrats).

::

    pragma solidity >=0.4.22 <0.6.0;

    contract Purchase {
        address public seller;

        modifier onlySeller() { // déclaration du modificateur
            require(
                msg.sender == seller,
                "Only seller can call this."
            );
            _;
        }

        function abort() public view onlySeller { // utilisation
            // ...
        }
    }

.. _structure-events:

Événements
==========

Les événements sont des interfaces pratiques avec les fonctionnalités de journalisation (logs) de l'EVM.

::

    pragma solidity >=0.4.21 <0.6.0;

    contract SimpleAuction {
        event HighestBidIncreased(address bidder, uint amount); // Event

        function bid() public payable {
            // ...
            emit HighestBidIncreased(msg.sender, msg.value); // Triggering event
        }
    }

Voir :ref:`events' dans la section contrats pour plus d'informations sur la façon dont les événements sont déclarés et peuvent être utilisés à partir d'une dapp.

.. _structure-struct-types:

Types Structure
===============

Les structures sont des types personnalisés qui peuvent regrouper plusieurs variables (voir
:ref:`structs` dans la section types).

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

Types Enum
==========

Les énumérations peuvent être utilisées pour créer des types personnalisés avec un ensemble fini de 'valeurs constantes' (voir :ref:`enums` dans la section types).

::

    pragma solidity >=0.4.0 <0.6.0;

    contract Purchase {
        enum State { Created, Locked, Inactive } // Enum
    }
