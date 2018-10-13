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

Modificateurs de Fonctions
==========================

Les modificateurs (``modifier``) de fonctions peuvent être utilisés pour modifier la sémantique des fonctions de manière déclarative (voir :ref:`modifiers` dans la section contrats).

::

    pragma solidity >=0.4.22 <0.6.0;

    contract Purchase {
        address public seller;

        modifier onlySeller() { // Modificateur
            require(
                msg.sender == seller,
                "Only seller can call this."
            );
            _;
        }

        function abort() public view onlySeller { // Utilisation du modificateur
            // ...
        }
    }

.. _structure-events:

Évènements
==========

Les 'ev`enements (``event``) sont une interface d'accès aux fonctionnalités de journalisation de l'EVM.

::

    pragma solidity >=0.4.21 <0.6.0;

    contract SimpleAuction {
        event HighestBidIncreased(address bidder, uint amount); // Event

        function bid() public payable {
            // ...
            emit HighestBidIncreased(msg.sender, msg.value); // Triggering event
        }
    }

Voir :ref:`events` dans la section contrats pour plus d'informations sur la façon dont les événements sont déclarés et peuvent être utilisés à partir d'une dapp.

.. _structure-struct-types:

Type Structure
==============

Les structures (``struct``) sont des types personnalisés qui peuvent regrouper plusieurs variables (voir :ref:`structs` dans la section Types).

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

Type Enum
=========

Les Enumérateurs (``enum``) peuvent être utilisés pour créer des types personnalisés avec un ensemble fini de 'valeurs constantes' (voir :ref:`enums` dans la section Types).

::

    pragma solidity >=0.4.0 <0.6.0;

    contract Purchase {
        enum State { Created, Locked, Inactive } // Enum
    }
