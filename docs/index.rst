Solidity
========

.. image:: logo.svg
    :width: 120px
    :alt: Solidity logo
    :align: center

Solidity est un langage haut-niveau, orienté objet dédié à l'implémentation de smart contracts. Les smart contracts (littéralement contrats intelligents) sont des programes qui régissent le comportement de comptes dans l'état d'Ethereum.

Solidity a été influencé par C++, Python et JavaScript et est conçu pour cibler la machine virtuelle Ethereum (EVM).

Solidity est statiquement typé, supporte l'héritage, les librairies et les bibliothèques, ainsi
que les types complexes définis par l'utilisateur parmi d'autres caractéristiques.

Avec Solidity, vous pouvez créer des contrats pour des usages tels que le vote, le crowdfunding, les enchères à l'aveugle,
et portefeuilles multi-signature.

Pour déploye des contrat, vous devriez utiliser la dernière version disponible ne téléchargement de Solidity. 
Ceci car des changements importants, des corrections de bugs et de nouvelles caractèristiques sont introduits  régulièrement. 

Nous utilison actuellement un numéro 0.x pour indiquer le changement rapide
 <https://semver.org/#spec-item-4>`_.

.. Avertissement::

  Solidity a récement introduit la version 0.6.x qavec beaucoup de chagements structurels. 
  Lisez bien la documentation:doc:`the full list <060-breaking-changes>`.

Documentation du langage
------------------------

Si vous n'êtes pas familié avec le concept de smart contracts, nous vous recommendons de commencer avec
:ref:`un exemple simple de smart contract <simple-smart-contract>` écrit en  Solidity. 
Quand vous serez prêts pour davantage de détails, nous vous recommandons de lire
:doc:`"Solidity par l'exemple" <solidity-by-example>` et
les sections "Description u langage" pour apprendre les concepts de base du langage. 


Pour une lecture plus avancée, essayez :ref:`the basics of blockchains <blockchain-basics>`
et les détails de  :ref:`Ethereum Virtual Machine <the-ethereum-virtual-machine>`.

.. Astuce::
  Rappelez-vous que vous pouvez toujours essayer les contrats `dans votre navigateur <https://remix.ethereum.org>`_ ! Remix est un IDE dans un navigateu
  permettant d'écrire des spart contracts Solidity, puis de déployer et exécuter les smart contracts. 
  Cela peut prendre du temps; soyez donc patients. 

.. warning::
    As humans write software, it can have bugs. You should follow established
    software development best-practices when writing your smart contracts, this
    includes code review, testing, audits, and correctness proofs. Smart contract
    users are sometimes more confident with code than their authors, and
    blockchains and smart contracts have their own unique issues to
    watch out for, so before working on production code, make sure you read the
    :ref:`security_considerations` section.

.. _translations:

Traductions
-----------

Cette documentation est traduite en plusieurs langues par des bénévoles de la communauté avec divers degrés d'exhaustivité et d'actualité. La version anglaise reste la référence.

* `French <http://solidity-fr.readthedocs.io>`_ (en cours)
* `Italian <https://github.com/damianoazzolini/solidity>`_ (en cours)
* `Japanese <https://solidity-jp.readthedocs.io>`_
* `Korean <http://solidity-kr.readthedocs.io>`_ (en cours)
* `Russian <https://github.com/ethereum/wiki/wiki/%5BRussian%5D-%D0%A0%D1%83%D0%BA%D0%BE%D0%B2%D0%BE%D0%B4%D1%81%D1%82%D0%B2%D0%BE-%D0%BF%D0%BE-Solidity>`_ (plutôt dépassée)
* `Simplified Chinese <http://solidity-cn.readthedocs.io>`_ (en cours)
* `Spanish <https://solidity-es.readthedocs.io>`_
* `Turkish <https://github.com/denizozzgur/Solidity_TR/blob/master/README.md>`_ (partielle)

Sommaire
========

:ref:`Keyword Index <genindex>`, :ref:`Search Page <search>`

.. toctree::
   :maxdepth: 2
   :caption: Les bases

   introduction-to-smart-contracts.rst
   installing-solidity.rst
   solidity-by-example.rst

.. toctree::
   :maxdepth: 2
   :caption: Description du langage

   layout-of-source-files.rst
   structure-of-a-contract.rst
   types.rst
   units-and-global-variables.rst
   control-structures.rst
   contracts.rst
   assembly.rst
   cheatsheet.rst
   grammar.rst

.. toctree::
   :maxdepth: 2
   :caption: Internals

   internals/layout_in_storage.rst
   internals/layout_in_memory.rst
   internals/layout_in_calldata.rst
   internals/variable_cleanup.rst
   internals/source_mappings.rst
   internals/optimiser.rst
   metadata.rst
   abi-spec.rst

.. toctree::
   :maxdepth: 2
   :caption: Ressources additionelles

   050-breaking-changes.rst
   060-breaking-changes.rst
   natspec-format.rst
   security-considerations.rst
   resources.rst
   using-the-compiler.rst
   yul.rst
   style-guide.rst
   common-patterns.rst
   bugs.rst
   contributing.rst
