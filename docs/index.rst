Solidity
========

Solidity est un langage haut-niveau, orienté objet dédié à l'implémentation de smart contracts. Les smart contracts (littéralement contrats intelligents) sont des programes qui régissent le comportement de comptes dans l'état d'Ethereum.


Solidity est un `curly-bracket language <https://en.wikipedia.org/wiki/List_of_programming_languages_by_type#Curly-bracket_languages>`_ (langage à accolades).
Solidity a été influencé par C++, Python et JavaScript et est conçu pour cibler
la machine virtuelle Ethereum (EVM).
Plus d'informations sur les langages qui ont inspiré Solidity dans la section :doc:`language influences <language-influences>`.

Solidity est statiquement typé, supporte l'héritage, les librairies et les bibliothèques, ainsi
que les types complexes définis par l'utilisateur parmi d'autres caractéristiques.

Avec Solidity, vous pouvez créer des contrats pour des usages tels que le vote, le crowdfunding, les enchères à l'aveugle,
et portefeuilles multi-signature.

Pour déployer des contrats, vous devriez utiliser la dernière version de Solidity publiée. Apart from exceptional cases, only the latest version receives
`security fixes <https://github.com/ethereum/solidity/security/policy#supported-versions>`_.
Ceci car des changements importants, des corrections de bugs et de nouvelles caractèristiques sont introduits régulièrement. We currently use
a 0.y.z version number `to indicate this fast pace of change <https://semver.org/#spec-item-4>`_.

Nous utilison actuellement un numéro 0.x pour indiquer le changement rapide
 <https://semver.org/#spec-item-4>`_.

.. Avertissement::

  Solidity a récement introduit la version 0.6.x avec beaucoup de changements structurels. 
  Lisez bien la documentation:doc:`the full list <060-breaking-changes>`.

Ideas for improving Solidity or this documentation are always welcome,
read our :doc:`contributors guide <contributing>` for more details.

.. Hint::

  You can download this documentation as PDF, HTML or Epub by clicking on the versions
  flyout menu in the bottom-left corner and selecting the preferred download format.


Getting Started
---------------

**1. Understand the Smart Contract Basics**

Si vous n'êtes pas familier avec le concept de smart contracts, nous vous recommendons de commencer avec
:ref:`un exemple simple de smart contract <simple-smart-contract>` écrit en  Solidity. 
Quand vous serez prêts pour davantage de détails, nous vous recommandons de lire
:doc:`"Solidity par l'exemple" <solidity-by-example>` et
les sections "Description du langage" pour apprendre les concepts de base du langage. 


Pour une lecture plus avancée, essayez :ref:`the basics of blockchains <blockchain-basics>`
et les détails de  :ref:`Ethereum Virtual Machine <the-ethereum-virtual-machine>`.

**2. Get to Know Solidity**

Once you are accustomed to the basics, we recommend you read the :doc:`"Solidity by Example" <solidity-by-example>`
and “Language Description” sections to understand the core concepts of the language.

**3. Install the Solidity Compiler**

There are various ways to install the Solidity compiler,
simply choose your preferred option and follow the steps outlined on the :ref:`installation page <installing-solidity>`.

.. Astuce::
  Rappelez-vous que vous pouvez toujours essayer les contrats `dans votre navigateur <https://remix.ethereum.org>`_ ! Remix est un IDE dans un navigateur
  permettant d'écrire des smart contracts Solidity, puis de déployer et exécuter les smart contracts. 
  Cela peut prendre du temps; soyez donc patients. 

.. warning::
    As humans write software, it can have bugs. You should follow established
    software development best-practices when writing your smart contracts. This
    includes code review, testing, audits, and correctness proofs. Smart contract
    users are sometimes more confident with code than their authors, and
    blockchains and smart contracts have their own unique issues to
    watch out for, so before working on production code, make sure you read the
    :ref:`security_considerations` section.

**4. Learn More**

If you want to learn more about building decentralized applications on Ethereum, the
`Ethereum Developer Resources <https://ethereum.org/en/developers/>`_
can help you with further general documentation around Ethereum, and a wide selection of tutorials,
tools and development frameworks.

If you have any questions, you can try searching for answers or asking on the
`Ethereum StackExchange <https://ethereum.stackexchange.com/>`_, or
our `Gitter channel <https://gitter.im/ethereum/solidity/>`_.

.. _translations:

Traductions
-----------

Cette documentation est traduite en plusieurs langues par des bénévoles de la communauté avec divers degrés d'exhaustivité et d'actualité. La version anglaise reste la référence.

.. note::

   We recently set up a new GitHub organization and translation workflow to help streamline the
   community efforts. Please refer to the `translation guide <https://github.com/solidity-docs/translation-guide>`_
   for information on how to contribute to the community translations moving forward.

* `French <http://solidity-fr.readthedocs.io>`_ (en cours)
* `Italian <https://github.com/damianoazzolini/solidity>`_ (en cours)
* `Japanese <https://solidity-jp.readthedocs.io>`_
* `Korean <http://solidity-kr.readthedocs.io>`_ (en cours)
* `Russian <https://github.com/ethereum/wiki/wiki/%5BRussian%5D-%D0%A0%D1%83%D0%BA%D0%BE%D0%B2%D0%BE%D0%B4%D1%81%D1%82%D0%B2%D0%BE-%D0%BF%D0%BE-Solidity>`_ (plutôt dépassée)
* `Simplified Chinese <https://learnblockchain.cn/docs/solidity/>`_ (en cours)
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
   :caption: Compiler

   using-the-compiler.rst
   analysing-compilation-output.rst
   ir-breaking-changes.rst

.. toctree::
   :maxdepth: 2
   :caption: Internals

   internals/layout_in_storage.rst
   internals/layout_in_memory.rst
   internals/layout_in_calldata.rst
   internals/variable_cleanup.rst
   internals/source_mappings.rst
   internals/optimizer.rst
   metadata.rst
   abi-spec.rst

.. toctree::
   :maxdepth: 2
   :caption: Ressources additionelles

   050-breaking-changes.rst
   060-breaking-changes.rst
   070-breaking-changes.rst
   080-breaking-changes.rst
   natspec-format.rst
   security-considerations.rst
   smtchecker.rst
   resources.rst
   path-resolution.rst
   yul.rst
   style-guide.rst
   common-patterns.rst
   bugs.rst
   contributing.rst
   brand-guide.rst
   language-influences.rst
