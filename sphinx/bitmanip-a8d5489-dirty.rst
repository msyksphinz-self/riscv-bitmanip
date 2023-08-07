======================================
RISC-V Bit-Manipulation ISA-extensions
======================================

:Date:   2023-08-07

This document is released under the `Creative Commons Attribution 4.0
International License <https://creativecommons.org/licenses/by/4.0/>`__.

It describes the BitManip Zba, Zbb, Zbc and Zbs extensions being
submitted for public review.

| Contributors to this specification (in alphabetical order) include:
| Jacob Bachmeyer, Allen Baum, Ari Ben, Alex Bradbury, Steven Braeger,
  Rogier Brussee, Michael Clark, Ken Dockser, Paul Donahue, Dennis
  Ferguson, Fabian Giesen, John Hauser, Robert Henry, Bruce Hoult,
  Po-wei Huang, Ben Marshall, Rex McCrary, Lee Moore, Jiří Moravec,
  Samuel Neves, Markus Oberhumer, Christopher Olson, Nils Pipenbrinck,
  Joseph Rahmeh, Xue Saw, Tommy Thorn, Philipp Tomsich, Avishai Tvila,
  Andrew Waterman, Thomas Wicki, and Claire Wolf.

We express our gratitude to everyone that contributed to, reviewed or
improved this specification through their comments and questions.

.. __ビットマニピュレーションabcs拡張の公開レビューと批准のためのグループ化:

ビットマニピュレーションa、b、c、s拡張の公開レビューと批准のためのグループ化
============================================================================

ビットマニピュレーション(bitmanip)拡張は、ベースとなるRISC-Vアーキテクチャに対するいくつかのコンポーネント拡張で構成され、コードサイズの削減、性能向上、エネルギー削減の組み合わせを提供することを目的としている。
これらの命令は一般的に使用されることを意図しているが、いくつかの命令は他の命令よりもある領域で有用である。
そのため、1つの大きな拡張ではなく、いくつかの小さなbitmanip拡張が提供されている。
これらの小さな拡張はそれぞれ共通の機能と使用例によってグループ化され、それぞれ独自のZb*拡張名を持っている。

各 bitmanip
拡張は、同じような目的を持ち、しばしば同じロジックを共有することができるいくつかの
bitmanip 命令のグループを含む。
いくつかの命令は1つの拡張子で利用可能であり、他の命令は複数の拡張子で利用可能である。
命令は、それらが現れる拡張に依存しないニーモニックとエンコーディングを持つ。
したがって、重複する命令を持つ拡張機能を実装する場合、ロジックやエンコーディングに冗長性はない。

bitmanip拡張はRV32とRV64用に定義されている。
ほとんどの命令はRV128と前方互換性があると予想される。
シフト即値命令は最大6ビットの即値フィールドを持つように定義されているが、RV128で必要とされる場合には、エンコード空間に7ビット目が用意されている。

.. __ワード命令:

ワード命令
==========

bitmanip拡張は、 *w* 付き命令 (
\_w_の前にドットがない)は入力の上位32ビットを無視し、最下位32ビットを符号付き値として演算し、符号をXLENに拡張した32ビットの符号付き結果を生成するというRV64の慣例に従っている。

接尾辞が *.uw* の Bitmanip 命令は、指定されたレジスタの最下位 32
ビットから抽出された符号なし 32 ビット値をオペランドとして持つ。
それ以外は、完全なXLEN演算を行う。

接尾辞 *.b* , *.h* , *.w* を持つbitmanip命令は、入力の最下位 8
ビット、16 ビット、32 ビット（それぞれ）のみを参照し、符号拡張された
XLEN幅の結果を生成する。
この結果は、特定の命令に基づいて符号拡張またはゼロ拡張される。

.. __命令セマンティクスのための疑似コード:

命令セマンティクスのための疑似コード
====================================

`Instructions (in alphabetical order) <#insns>`__
で記述される各命令のセマンティクスは、SAILの構文で記述される。

.. __extensions:

Extensions
==========

パブリック・レビューとして後悔されたbitmanipの最初のグループは以下である。

-  `Address generation instructions <#zba>`__

-  `Basic bit-manipulation <#zbb>`__

-  `Carry-less multiplication <#zbc>`__

-  `Single-bit instructions <#zbs>`__

以下は、これらの拡張に含まれるすべての命令(および擬似命令)のリストとそのマッピングの一覧である:

+----+----+-----------------+---------------------------+---+---+---+---+
| RV | RV | Mnemonic        | Instruction               | Z | Z | Z | Z |
| 32 | 64 |                 |                           | b | b | b | b |
|    |    |                 |                           | a | b | c | s |
+====+====+=================+===========================+===+===+===+===+
|    | ✓  | add.uw *rd*,    | `Add unsigned             | ✓ |   |   |   |
|    |    | *rs1*, *rs2*    | word <#insns-add_uw>`__   |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | andn *rd*,      | `AND with inverted        |   | ✓ |   |   |
|    |    | *rs1*, *rs2*    | operand <#insns-andn>`__  |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | clmul *rd*,     | `Carry-less multiply      |   |   | ✓ |   |
|    |    | *rs1*, *rs2*    | (lo                       |   |   |   |   |
|    |    |                 | w-part) <#insns-clmul>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | clmulh *rd*,    | `Carry-less multiply      |   |   | ✓ |   |
|    |    | *rs1*, *rs2*    | (high                     |   |   |   |   |
|    |    |                 | -part) <#insns-clmulh>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | clmulr *rd*,    | `Carry-less multiply      |   |   | ✓ |   |
|    |    | *rs1*, *rs2*    | (rev                      |   |   |   |   |
|    |    |                 | ersed) <#insns-clmulr>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | clz *rd*, *rs*  | `Count leading zero       |   | ✓ |   |   |
|    |    |                 | bits <#insns-clz>`__      |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
|    | ✓  | clzw *rd*, *rs* | `Count leading zero bits  |   | ✓ |   |   |
|    |    |                 | in word <#insns-clzw>`__  |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | cpop *rd*, *rs* | `Count set                |   | ✓ |   |   |
|    |    |                 | bits <#insns-cpop>`__     |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
|    | ✓  | cpopw *rd*,     | `Count set bits in        |   | ✓ |   |   |
|    |    | *rs*            | word <#insns-cpopw>`__    |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | ctz *rd*, *rs*  | `Count trailing zero      |   | ✓ |   |   |
|    |    |                 | bits <#insns-ctz>`__      |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
|    | ✓  | ctzw *rd*, *rs* | `Count trailing zero bits |   | ✓ |   |   |
|    |    |                 | in word <#insns-ctzw>`__  |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | max *rd*,       | `Maximum <#insns-max>`__  |   | ✓ |   |   |
|    |    | *rs1*, *rs2*    |                           |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | maxu *rd*,      | `Unsigned                 |   | ✓ |   |   |
|    |    | *rs1*, *rs2*    | maximum <#insns-maxu>`__  |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | min *rd*,       | `Minimum <#insns-min>`__  |   | ✓ |   |   |
|    |    | *rs1*, *rs2*    |                           |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | minu *rd*,      | `Unsigned                 |   | ✓ |   |   |
|    |    | *rs1*, *rs2*    | minimum <#insns-minu>`__  |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | orc.b *rd*,     | `Bitwise OR-Combine, byte |   | ✓ |   |   |
|    |    | *rs1*, *rs2*    | granule <#insns-orc_b>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | orn *rd*,       | `OR with inverted         |   | ✓ |   |   |
|    |    | *rs1*, *rs2*    | operand <#insns-orn>`__   |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | rev8 *rd*, *rs* | `Byte-reverse             |   | ✓ |   |   |
|    |    |                 | register <#insns-rev8>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | rol *rd*,       | `Rotate left              |   | ✓ |   |   |
|    |    | *rs1*, *rs2*    | (                         |   |   |   |   |
|    |    |                 | Register) <#insns-rol>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
|    | ✓  | rolw *rd*,      | `Rotate Left Word         |   | ✓ |   |   |
|    |    | *rs1*, *rs2*    | (R                        |   |   |   |   |
|    |    |                 | egister) <#insns-rolw>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | ror *rd*,       | `Rotate right             |   | ✓ |   |   |
|    |    | *rs1*, *rs2*    | (                         |   |   |   |   |
|    |    |                 | Register) <#insns-ror>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | rori *rd*,      | `Rotate right             |   | ✓ |   |   |
|    |    | *rs1*, *shamt*  | (Im                       |   |   |   |   |
|    |    |                 | mediate) <#insns-rori>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
|    | ✓  | roriw *rd*,     | `Rotate right Word        |   | ✓ |   |   |
|    |    | *rs1*, *shamt*  | (Imm                      |   |   |   |   |
|    |    |                 | ediate) <#insns-roriw>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
|    | ✓  | rorw *rd*,      | `Rotate right Word        |   | ✓ |   |   |
|    |    | *rs1*, *rs2*    | (R                        |   |   |   |   |
|    |    |                 | egister) <#insns-rorw>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | bclr *rd*,      | `Single-Bit Clear         |   |   |   | ✓ |
|    |    | *rs1*, *rs2*    | (R                        |   |   |   |   |
|    |    |                 | egister) <#insns-bclr>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | bclri *rd*,     | `Single-Bit Clear         |   |   |   | ✓ |
|    |    | *rs1*, *imm*    | (Imm                      |   |   |   |   |
|    |    |                 | ediate) <#insns-bclri>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | bext *rd*,      | `Single-Bit Extract       |   |   |   | ✓ |
|    |    | *rs1*, *rs2*    | (R                        |   |   |   |   |
|    |    |                 | egister) <#insns-bext>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | bexti *rd*,     | `Single-Bit Extract       |   |   |   | ✓ |
|    |    | *rs1*, *imm*    | (Imm                      |   |   |   |   |
|    |    |                 | ediate) <#insns-bexti>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | binv *rd*,      | `Single-Bit Invert        |   |   |   | ✓ |
|    |    | *rs1*, *rs2*    | (R                        |   |   |   |   |
|    |    |                 | egister) <#insns-binv>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | binvi *rd*,     | `Single-Bit Invert        |   |   |   | ✓ |
|    |    | *rs1*, *imm*    | (Imm                      |   |   |   |   |
|    |    |                 | ediate) <#insns-binvi>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | bset *rd*,      | `Single-Bit Set           |   |   |   | ✓ |
|    |    | *rs1*, *rs2*    | (R                        |   |   |   |   |
|    |    |                 | egister) <#insns-bset>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | bseti *rd*,     | `Single-Bit Set           |   |   |   | ✓ |
|    |    | *rs1*, *imm*    | (Imm                      |   |   |   |   |
|    |    |                 | ediate) <#insns-bseti>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | sext.b *rd*,    | `Sign-extend              |   | ✓ |   |   |
|    |    | *rs*            | byte <#insns-sext_b>`__   |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | sext.h *rd*,    | `Sign-extend              |   | ✓ |   |   |
|    |    | *rs*            | ha                        |   |   |   |   |
|    |    |                 | lfword <#insns-sext_h>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | sh1add *rd*,    | `Shift left by 1 and      | ✓ |   |   |   |
|    |    | *rs1*, *rs2*    | add <#insns-sh1add>`__    |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
|    | ✓  | sh1add.uw *rd*, | `Shift unsigned word left | ✓ |   |   |   |
|    |    | *rs1*, *rs2*    | by 1 and                  |   |   |   |   |
|    |    |                 | add <#insns-sh1add_uw>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | sh2add *rd*,    | `Shift left by 2 and      | ✓ |   |   |   |
|    |    | *rs1*, *rs2*    | add <#insns-sh2add>`__    |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
|    | ✓  | sh2add.uw *rd*, | `Shift unsigned word left | ✓ |   |   |   |
|    |    | *rs1*, *rs2*    | by 2 and                  |   |   |   |   |
|    |    |                 | add <#insns-sh2add_uw>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | sh3add *rd*,    | `Shift left by 3 and      | ✓ |   |   |   |
|    |    | *rs1*, *rs2*    | add <#insns-sh3add>`__    |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
|    | ✓  | sh3add.uw *rd*, | `Shift unsigned word left | ✓ |   |   |   |
|    |    | *rs1*, *rs2*    | by 3 and                  |   |   |   |   |
|    |    |                 | add <#insns-sh3add_uw>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
|    | ✓  | slli.uw *rd*,   | `Shift-left unsigned word | ✓ |   |   |   |
|    |    | *rs1*, *imm*    | (Immed                    |   |   |   |   |
|    |    |                 | iate) <#insns-slli_uw>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | xnor *rd*,      | `Exclusive                |   | ✓ |   |   |
|    |    | *rs1*, *rs2*    | NOR <#insns-xnor>`__      |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
| ✓  | ✓  | zext.h *rd*,    | `Zero-extend              |   | ✓ |   |   |
|    |    | *rs*            | ha                        |   |   |   |   |
|    |    |                 | lfword <#insns-zext_h>`__ |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+
|    | ✓  | zext.w *rd*,    | `Add unsigned             | ✓ |   |   |   |
|    |    | *rs*            | word <#insns-add_uw>`__   |   |   |   |   |
+----+----+-----------------+---------------------------+---+---+---+---+

.. _zba:

Zba extension
-------------

.. note::

   The Zba extension is frozen.

Zba命令は、符号なしワードサイズとXLENサイズの両方のインデックスを使用して、基本タイプ(ハーフワード、ワード、ダブルワード)の配列にインデックスを付けるアドレスの生成を高速化するために使用できる。

シフト命令と加算命令で1、2、3の左シフトを行うのは、実際のコードで一般的であり、単純な加算器以上の最小限の追加ハードウェアで実装できるからである。
これにより、実装におけるクリティカル・パスが長くなるのを避けることができる。

シフト命令と加算命令の最大左シフト数は3に制限されているが、(ベースISAの)slli
命令を使用すると、より広い要素の配列にインデックスを付けるために同様のシフトを実行できる。
このサブ拡張で追加されたslli.uwは、インデックスを符号なしワードとして解釈する場合に使用できる。

Zba拡張は以下の命令で構成されている:

+----+----+-------------------+----------------------------------------+
| RV | RV | Mnemonic          | Instruction                            |
| 32 | 64 |                   |                                        |
+====+====+===================+========================================+
|    | ✓  | add.uw *rd*,      | `Add unsigned word <#insns-add_uw>`__  |
|    |    | *rs1*, *rs2*      |                                        |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | sh1add *rd*,      | `Shift left by 1 and                   |
|    |    | *rs1*, *rs2*      | add <#insns-sh1add>`__                 |
+----+----+-------------------+----------------------------------------+
|    | ✓  | sh1add.uw *rd*,   | `Shift unsigned word left by 1 and     |
|    |    | *rs1*, *rs2*      | add <#insns-sh1add_uw>`__              |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | sh2add *rd*,      | `Shift left by 2 and                   |
|    |    | *rs1*, *rs2*      | add <#insns-sh2add>`__                 |
+----+----+-------------------+----------------------------------------+
|    | ✓  | sh2add.uw *rd*,   | `Shift unsigned word left by 2 and     |
|    |    | *rs1*, *rs2*      | add <#insns-sh2add_uw>`__              |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | sh3add *rd*,      | `Shift left by 3 and                   |
|    |    | *rs1*, *rs2*      | add <#insns-sh3add>`__                 |
+----+----+-------------------+----------------------------------------+
|    | ✓  | sh3add.uw *rd*,   | `Shift unsigned word left by 3 and     |
|    |    | *rs1*, *rs2*      | add <#insns-sh3add_uw>`__              |
+----+----+-------------------+----------------------------------------+
|    | ✓  | slli.uw *rd*,     | `Shift-left unsigned word              |
|    |    | *rs1*, *imm*      | (Immediate) <#insns-slli_uw>`__        |
+----+----+-------------------+----------------------------------------+
|    | ✓  | zext.w *rd*, *rs* | `Add unsigned word <#insns-add_uw>`__  |
+----+----+-------------------+----------------------------------------+

.. _zbb:

Zbb: Basic bit-manipulation
---------------------------

.. note::

   Zbb拡張はFrozen状態である。

.. __否定付き論理演算命令:

否定付き論理演算命令
~~~~~~~~~~~~~~~~~~~~

+----+----+-------------------+----------------------------------------+
| RV | RV | Mnemonic          | Instruction                            |
| 32 | 64 |                   |                                        |
+====+====+===================+========================================+
| ✓  | ✓  | andn *rd*, *rs1*, | `AND with inverted                     |
|    |    | *rs2*             | operand <#insns-andn>`__               |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | orn *rd*, *rs1*,  | `OR with inverted                      |
|    |    | *rs2*             | operand <#insns-orn>`__                |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | xnor *rd*, *rs1*, | `Exclusive NOR <#insns-xnor>`__        |
|    |    | *rs2*             |                                        |
+----+----+-------------------+----------------------------------------+

.. note::

   The Logical with Negate instructions can be implemented by inverting
   the *rs2* inputs to the base-required AND, OR, and XOR logic
   instructions. In some implementations, the inverter on rs2 used for
   subtraction can be reused for this purpose.

.. __leadingtrailing_ゼロビットカウント命令:

Leading/Trailing ゼロビットカウント命令
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

+----+----+-------------------+----------------------------------------+
| RV | RV | Mnemonic          | Instruction                            |
| 32 | 64 |                   |                                        |
+====+====+===================+========================================+
| ✓  | ✓  | clz *rd*, *rs*    | `Count leading zero                    |
|    |    |                   | bits <#insns-clz>`__                   |
+----+----+-------------------+----------------------------------------+
|    | ✓  | clzw *rd*, *rs*   | `Count leading zero bits in            |
|    |    |                   | word <#insns-clzw>`__                  |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | ctz *rd*, *rs*    | `Count trailing zero                   |
|    |    |                   | bits <#insns-ctz>`__                   |
+----+----+-------------------+----------------------------------------+
|    | ✓  | ctzw *rd*, *rs*   | `Count trailing zero bits in           |
|    |    |                   | word <#insns-ctzw>`__                  |
+----+----+-------------------+----------------------------------------+

.. __pop_count命令:

Pop Count命令
~~~~~~~~~~~~~

これらの命令はセットされている(ビットが1)の数を数える。
これは一般的にPopulation Countと呼ばれている。

+----+----+-------------------+----------------------------------------+
| RV | RV | Mnemonic          | Instruction                            |
| 32 | 64 |                   |                                        |
+====+====+===================+========================================+
| ✓  | ✓  | cpop *rd*, *rs*   | `Count set bits <#insns-cpop>`__       |
+----+----+-------------------+----------------------------------------+
|    | ✓  | cpopw *rd*, *rs*  | `Count set bits in                     |
|    |    |                   | word <#insns-cpopw>`__                 |
+----+----+-------------------+----------------------------------------+

.. __整数最大値最小値命令:

整数最大値・最小値命令
~~~~~~~~~~~~~~~~~~~~~~

整数最大値・最小値命令はR-typeの算術演算命令であり、
2つのオペランドの最大値・最小値を返す。

+----+----+-------------------+----------------------------------------+
| RV | RV | Mnemonic          | Instruction                            |
| 32 | 64 |                   |                                        |
+====+====+===================+========================================+
| ✓  | ✓  | max *rd*, *rs1*,  | `Maximum <#insns-max>`__               |
|    |    | *rs2*             |                                        |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | maxu *rd*, *rs1*, | `Unsigned maximum <#insns-maxu>`__     |
|    |    | *rs2*             |                                        |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | min *rd*, *rs1*,  | `Minimum <#insns-min>`__               |
|    |    | *rs2*             |                                        |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | minu *rd*, *rs1*, | `Unsigned minimum <#insns-minu>`__     |
|    |    | *rs2*             |                                        |
+----+----+-------------------+----------------------------------------+

.. __符号拡張ゼロ拡張命令:

符号拡張・ゼロ拡張命令
~~~~~~~~~~~~~~~~~~~~~~

これらの命令はソース・レジスタの最下位8ビット、16ビット、32ビットを符号拡張もしくはゼロ拡張する。

これらの命令は、8ビットおよび16ビットのゼロ拡張時は
``slli rD,rs,(XLEN-<size>) + srli`` 命令、
16ビットおよび32ビットの符号拡張時は ``slli + srai``
という一般的なイディオムとして置き換えることができる。

+----+----+-------------------+----------------------------------------+
| RV | RV | Mnemonic          | Instruction                            |
| 32 | 64 |                   |                                        |
+====+====+===================+========================================+
| ✓  | ✓  | sext.b *rd*, *rs* | `Sign-extend byte <#insns-sext_b>`__   |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | sext.h *rd*, *rs* | `Sign-extend                           |
|    |    |                   | halfword <#insns-sext_h>`__            |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | zext.h *rd*, *rs* | `Zero-extend                           |
|    |    |                   | halfword <#insns-zext_h>`__            |
+----+----+-------------------+----------------------------------------+

.. __ローテート命令:

ローテート命令
~~~~~~~~~~~~~~

ビット単位の回転命令は、基本仕様のシフト論理演算に似ている。
ただし、シフトがゼロをシフトするのに対し、ローテート命令は値の反対側にシフトされたビットをシフトする。
このような操作は’循環シフト’とも呼ばれる。

+----+----+-------------------+----------------------------------------+
| RV | RV | Mnemonic          | Instruction                            |
| 32 | 64 |                   |                                        |
+====+====+===================+========================================+
| ✓  | ✓  | rol *rd*, *rs1*,  | `Rotate left                           |
|    |    | *rs2*             | (Register) <#insns-rol>`__             |
+----+----+-------------------+----------------------------------------+
|    | ✓  | rolw *rd*, *rs1*, | `Rotate Left Word                      |
|    |    | *rs2*             | (Register) <#insns-rolw>`__            |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | ror *rd*, *rs1*,  | `Rotate right                          |
|    |    | *rs2*             | (Register) <#insns-ror>`__             |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | rori *rd*, *rs1*, | `Rotate right                          |
|    |    | *shamt*           | (Immediate) <#insns-rori>`__           |
+----+----+-------------------+----------------------------------------+
|    | ✓  | roriw *rd*,       | `Rotate right Word                     |
|    |    | *rs1*, *shamt*    | (Immediate) <#insns-roriw>`__          |
+----+----+-------------------+----------------------------------------+
|    | ✓  | rorw *rd*, *rs1*, | `Rotate right Word                     |
|    |    | *rs2*             | (Register) <#insns-rorw>`__            |
+----+----+-------------------+----------------------------------------+

.. note::

   The rotate instructions were included to replace a common
   four-instruction sequence to achieve the same effect (neg; sll/srl;
   srl/sll; or)

.. __or組み合わせ命令:

OR組み合わせ命令
~~~~~~~~~~~~~~~~

**orc.b** は、結果 *rd* の各バイトのビットを、 *rs* の各バ
イト内のビットがセットされていなければすべてゼロに、 *rs* の各バ
イト内のビットがセットされていればすべて1にセットする。

使用例としては、 **strlen** や **strcpy** のような文字列処理関数がある。
ワード内の非ゼロのバイトのセットビットをカウントすることで、終端ゼロバイトをテストすることができる。

+----+----+-------------------+----------------------------------------+
| RV | RV | Mnemonic          | Instruction                            |
| 32 | 64 |                   |                                        |
+====+====+===================+========================================+
| ✓  | ✓  | orc.b *rd*, *rs*  | `Bitwise OR-Combine, byte              |
|    |    |                   | granule <#insns-orc_b>`__              |
+----+----+-------------------+----------------------------------------+

.. __バイト逆転命令:

バイト逆転命令
~~~~~~~~~~~~~~

**rev8** 命令は、 *rs* の倍との順序を逆転させる。

+----+----+-------------------+----------------------------------------+
| RV | RV | Mnemonic          | Instruction                            |
| 32 | 64 |                   |                                        |
+====+====+===================+========================================+
| ✓  | ✓  | rev8 *rd*, *rs*   | `Byte-reverse                          |
|    |    |                   | register <#insns-rev8>`__              |
+----+----+-------------------+----------------------------------------+

.. _zbc:

Zbc: キャリー無し乗算命令
-------------------------

.. note::

   Zbc拡張はFrozen状態である。

キャリー無し乗算はGF(2)上の多項式環における乗算である。

**clmul** はキャリーレス積の下半分を生成し、 **clmulh** は 2✕XLEN
キャリーレス積の上半分を生成する。

**clmulr** は 2✕XLEN キャリーレス積のビット 2✕XLEN-2:XLEN-1 を生成する。

+----+----+-------------------+----------------------------------------+
| RV | RV | Mnemonic          | Instruction                            |
| 32 | 64 |                   |                                        |
+====+====+===================+========================================+
| ✓  | ✓  | clmul *rd*,       | `Carry-less multiply                   |
|    |    | *rs1*, *rs2*      | (low-part) <#insns-clmul>`__           |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | clmulh *rd*,      | `Carry-less multiply                   |
|    |    | *rs1*, *rs2*      | (high-part) <#insns-clmulh>`__         |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | clmulr *rd*,      | `Carry-less multiply                   |
|    |    | *rs1*, *rs2*      | (reversed) <#insns-clmulr>`__          |
+----+----+-------------------+----------------------------------------+

.. _zbs:

Zbs: 単一ビット命令
-------------------

.. note::

   Zbc拡張はFrozen状態である。

シングルビット命令は、レジスタの単一ビットをセット、クリア、反転、または抽出するメカニズムを提供する。
ビットはインデックスで指定する。

+----+----+-------------------+----------------------------------------+
| RV | RV | Mnemonic          | Instruction                            |
| 32 | 64 |                   |                                        |
+====+====+===================+========================================+
| ✓  | ✓  | bclr *rd*, *rs1*, | `Single-Bit Clear                      |
|    |    | *rs2*             | (Register) <#insns-bclr>`__            |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | bclri *rd*,       | `Single-Bit Clear                      |
|    |    | *rs1*, *imm*      | (Immediate) <#insns-bclri>`__          |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | bext *rd*, *rs1*, | `Single-Bit Extract                    |
|    |    | *rs2*             | (Register) <#insns-bext>`__            |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | bexti *rd*,       | `Single-Bit Extract                    |
|    |    | *rs1*, *imm*      | (Immediate) <#insns-bexti>`__          |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | binv *rd*, *rs1*, | `Single-Bit Invert                     |
|    |    | *rs2*             | (Register) <#insns-binv>`__            |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | binvi *rd*,       | `Single-Bit Invert                     |
|    |    | *rs1*, *imm*      | (Immediate) <#insns-binvi>`__          |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | bset *rd*, *rs1*, | `Single-Bit Set                        |
|    |    | *rs2*             | (Register) <#insns-bset>`__            |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | bseti *rd*,       | `Single-Bit Set                        |
|    |    | *rs1*, *imm*      | (Immediate) <#insns-bseti>`__          |
+----+----+-------------------+----------------------------------------+

.. _zbkc:

Zbkc: 暗号向けキャリーレス乗算
------------------------------

.. note::

   Zbkc拡張はFrozen状態である。

キャリーレス乗算は、GF(2)上の多項式環における乗算である。
これはいくつかの暗号ワークロード、特にAES-GCM認証暗号化スキームにおいて重要な演算である。
この拡張は、このワークロードの一部であるGHASH演算を効率的に実装するために必要な命令のみを提供する。

+----+----+-------------------+----------------------------------------+
| RV | RV | Mnemonic          | Instruction                            |
| 32 | 64 |                   |                                        |
+====+====+===================+========================================+
| ✓  | ✓  | clmul *rd*,       | `Carry-less multiply                   |
|    |    | *rs1*, *rs2*      | (low-part) <#insns-clmul>`__           |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | clmulh *rd*,      | `Carry-less multiply                   |
|    |    | *rs1*, *rs2*      | (high-part) <#insns-clmulh>`__         |
+----+----+-------------------+----------------------------------------+

.. _zbkx:

Zbkx: クロスバ組み合わせ命令
----------------------------

.. note::

   Zbkx拡張はFrozen状態である。

これらの命令は、汎用レジスタ内の4ビットと8ビットの要素に対して"ルックアップテーブル"を実装する。
*rs1* はNビット・ワードのベクトルとして使用され、 *rs2* は *rs1*
へのNビット・インデックスのベクトルとして使用される。 *rs1* の要素は、
*rs2* のインデックス付き要素で置き換えられる。 *rs2*
へのインデックスが範囲外の場合はゼロとなる。

これらの命令は、Nビット対Nビットのブーリアン演算を表現したり、実行レイテンシが演算対象の(秘密)データに依存しないような、
秘密に依存するメモリアクセス(特にSBox)を持つ暗号コードを実装したりするのに便利である。

+----+----+-------------------+----------------------------------------+
| RV | RV | Mnemonic          | Instruction                            |
| 32 | 64 |                   |                                        |
+====+====+===================+========================================+
| ✓  | ✓  | xperm.n *rd*,     | `Crossbar permutation                  |
|    |    | *rs1*, *rs2*      | (nibbles) <#insns-xpermn>`__           |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | xperm.b *rd*,     | `Crossbar permutation                  |
|    |    | *rs1*, *rs2*      | (bytes) <#insns-xpermb>`__             |
+----+----+-------------------+----------------------------------------+

.. _zbkb:

Zbkb: 暗号化向けビット操作命令
------------------------------

.. note::

   Zbkb拡張はFrozen状態である。

この拡張には、暗号ワークロードの実装の基本となる共通動作のための命令が含まれている。

+----+----+-------------------+----------------------------------------+
| RV | RV | Mnemonic          | Instruction                            |
| 32 | 64 |                   |                                        |
+====+====+===================+========================================+
| ✓  | ✓  | rol               | `Rotate left                           |
|    |    |                   | (Register) <#insns-rol>`__             |
+----+----+-------------------+----------------------------------------+
|    | ✓  | rolw              | `Rotate Left Word                      |
|    |    |                   | (Register) <#insns-rolw>`__            |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | ror               | `Rotate right                          |
|    |    |                   | (Register) <#insns-ror>`__             |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | rori              | `Rotate right                          |
|    |    |                   | (Immediate) <#insns-rori>`__           |
+----+----+-------------------+----------------------------------------+
|    | ✓  | roriw             | `Rotate right Word                     |
|    |    |                   | (Immediate) <#insns-roriw>`__          |
+----+----+-------------------+----------------------------------------+
|    | ✓  | rorw              | `Rotate right Word                     |
|    |    |                   | (Register) <#insns-rorw>`__            |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | andn              | `AND with inverted                     |
|    |    |                   | operand <#insns-andn>`__               |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | orn               | `OR with inverted                      |
|    |    |                   | operand <#insns-orn>`__                |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | xnor              | `Exclusive NOR <#insns-xnor>`__        |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | pack              | `Pack low halves of                    |
|    |    |                   | registers <#insns-pack>`__             |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | packh             | `Pack low bytes of                     |
|    |    |                   | registers <#insns-packh>`__            |
+----+----+-------------------+----------------------------------------+
|    | ✓  | packw             | `Pack low 16-bits of registers         |
|    |    |                   | (RV64) <#insns-packw>`__               |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | rev.b             | `Reverse bits in                       |
|    |    |                   | bytes <#insns-revb>`__                 |
+----+----+-------------------+----------------------------------------+
| ✓  | ✓  | rev8              | `Byte-reverse                          |
|    |    |                   | register <#insns-rev8>`__              |
+----+----+-------------------+----------------------------------------+
| ✓  |    | zip               | `Bit interleave <#insns-zip>`__        |
+----+----+-------------------+----------------------------------------+
| ✓  |    | unzip             | `Bit deinterleave <#insns-unzip>`__    |
+----+----+-------------------+----------------------------------------+

.. _insns:

Instructions (in alphabetical order)
====================================

.. _insns-add_uw:

add.uw
------

Synopsis
   Add unsigned word

Mnemonic
   add.uw *rd*, *rs1*, *rs2*

Pseudoinstructions
   zext.w *rd*, *rs1* → add.uw *rd*, *rs1*, zero

Encoding

::

   Failed to generate image: Could not find the 'WaveDromEditor' executable in PATH; add it to the PATH or specify its location using the 'WaveDromEditor' document attribute
   {reg:[
       { bits:  7, name: 0x3b, attr: ['OP-32'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x0, attr: ['ADD.UW'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x04, attr: ['ADD.UW'] },
   ]}

Description
   This instruction performs an XLEN-wide addition between *rs2* and the
   zero-extended least-significant word of *rs1*.

Operation

.. code:: sail

   let base = X(rs2);
   let index = EXTZ(X(rs1)[31..0]);

   X(rd) = base + index;

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zba (`Address generation          | 0.93            | Frozen          |
| instructions <#zba>`__)           |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-andn:

andn
----

Synopsis
   AND with inverted operand

Mnemonic
   andn *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x33, attr: ['OP'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x7, attr: ['ANDN']},
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x20, attr: ['ANDN'] },
   ]}

Description
   This instruction performs the bitwise logical AND operation between
   *rs1* and the bitwise inversion of *rs2*.

Operation

.. code:: sail

   X(rd) = X(rs1) & ~X(rs2);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+
| Zbkb (`Bit-manipulation for       | v0.9.4          | Frozen          |
| Cryptography <#zbkb>`__)          |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-bclr:

bclr
----

Synopsis
   Single-Bit Clear (Register)

Mnemonic
   bclr *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x33, attr: ['OP'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['BCLR'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x24, attr: ['BCLR/BEXT'] },
   ]}

Description
   This instruction returns *rs1* with a single bit cleared at the index
   specified in *rs2*. The index is read from the lower log2(XLEN) bits
   of *rs2*.

Operation

.. code:: sail

   let index = X(rs2) & (XLEN - 1);
   X(rd) = X(rs1) & ~(1 << index)

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbs (`Single-bit                  | 0.93            | Frozen          |
| instructions <#zbs>`__)           |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-bclri:

bclri
-----

Synopsis
   Single-Bit Clear (Immediate)

Mnemonic
   bclri *rd*, *rs1*, *shamt*

Encoding (RV32)

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x13, attr: ['OP-IMM'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['BCLRI'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'shamt' },
       { bits:  7, name: 0x24, attr: ['BCLRI'] },
   ]}

Encoding (RV64)

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x13, attr: ['OP-IMM'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['BCLRI'] },
       { bits:  5, name: 'rs1' },
       { bits:  6, name: 'shamt' },
       { bits:  6, name: 0x12, attr: ['BCLRI'] },
   ]}

Description
   This instruction returns *rs1* with a single bit cleared at the index
   specified in *shamt*. The index is read from the lower log2(XLEN)
   bits of *shamt*. For RV32, the encodings corresponding to shamt[5]=1
   are reserved.

Operation

.. code:: sail

   let index = shamt & (XLEN - 1);
   X(rd) = X(rs1) & ~(1 << index)

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbs (`Single-bit                  | 0.93            | Frozen          |
| instructions <#zbs>`__)           |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-bext:

bext
----

Synopsis
   Single-Bit Extract (Register)

Mnemonic
   bext *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x33, attr: ['OP'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x5, attr: ['BEXT'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x24, attr: ['BCLR/BEXT'] },
   ]}

Description
   This instruction returns a single bit extracted from *rs1* at the
   index specified in *rs2*. The index is read from the lower log2(XLEN)
   bits of *rs2*.

Operation

.. code:: sail

   let index = X(rs2) & (XLEN - 1);
   X(rd) = (X(rs1) >> index) & 1;

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbs (`Single-bit                  | 0.93            | Frozen          |
| instructions <#zbs>`__)           |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-bexti:

bexti
-----

Synopsis
   Single-Bit Extract (Immediate)

Mnemonic
   bexti *rd*, *rs1*, *shamt*

Encoding (RV32)

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x13, attr: ['OP-IMM'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x5, attr: ['BEXTI'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'shamt' },
       { bits:  7, name: 0x24, attr: ['BEXTI/BCLRI'] },
   ]}

Encoding (RV64)

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x13, attr: ['OP-IMM'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x5, attr: ['BEXTI'] },
       { bits:  5, name: 'rs1' },
       { bits:  6, name: 'shamt' },
       { bits:  6, name: 0x12, attr: ['BEXTI/BCLRI'] },
   ]}

Description
   This instruction returns a single bit extracted from *rs1* at the
   index specified in *rs2*. The index is read from the lower log2(XLEN)
   bits of *shamt*. For RV32, the encodings corresponding to shamt[5]=1
   are reserved.

Operation

.. code:: sail

   let index = shamt & (XLEN - 1);
   X(rd) = (X(rs1) >> index) & 1;

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbs (`Single-bit                  | 0.93            | Frozen          |
| instructions <#zbs>`__)           |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-binv:

binv
----

Synopsis
   Single-Bit Invert (Register)

Mnemonic
   binv *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x33, attr: ['OP'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['BINV'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x34, attr: ['BINV'] },
   ]}

Description
   This instruction returns *rs1* with a single bit inverted at the
   index specified in *rs2*. The index is read from the lower log2(XLEN)
   bits of *rs2*.

Operation

.. code:: sail

   let index = X(rs2) & (XLEN - 1);
   X(rd) = X(rs1) ^ (1 << index)

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbs (`Single-bit                  | 0.93            | Frozen          |
| instructions <#zbs>`__)           |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-binvi:

binvi
-----

Synopsis
   Single-Bit Invert (Immediate)

Mnemonic
   binvi *rd*, *rs1*, *shamt*

Encoding (RV32)

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x13, attr: ['OP-IMM'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['BINV'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'shamt' },
       { bits:  7, name: 0x34, attr: ['BINVI'] },
   ]}

Encoding (RV64)

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x13, attr: ['OP-IMM'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['BINV'] },
       { bits:  5, name: 'rs1' },
       { bits:  6, name: 'shamt' },
       { bits:  6, name: 0x1a, attr: ['BINVI'] },
   ]}

Description
   This instruction returns *rs1* with a single bit inverted at the
   index specified in *shamt*. The index is read from the lower
   log2(XLEN) bits of *shamt*. For RV32, the encodings corresponding to
   shamt[5]=1 are reserved.

Operation

.. code:: sail

   let index = shamt & (XLEN - 1);
   X(rd) = X(rs1) ^ (1 << index)

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbs (`Single-bit                  | 0.93            | Frozen          |
| instructions <#zbs>`__)           |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-bset:

bset
----

Synopsis
   Single-Bit Set (Register)

Mnemonic
   bset *rd*, *rs1*,\ *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x33, attr: ['OP'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['BSET'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x14, attr: ['BSET'] },
   ]}

Description
   This instruction returns *rs1* with a single bit set at the index
   specified in *rs2*. The index is read from the lower log2(XLEN) bits
   of *rs2*.

Operation

.. code:: sail

   let index = X(rs2) & (XLEN - 1);
   X(rd) = X(rs1) | (1 << index)

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbs (`Single-bit                  | 0.93            | Frozen          |
| instructions <#zbs>`__)           |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-bseti:

bseti
-----

Synopsis
   Single-Bit Set (Immediate)

Mnemonic
   bseti *rd*, *rs1*,\ *shamt*

Encoding (RV32)

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x13, attr: ['OP-IMM'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['BSETI'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'shamt' },
       { bits:  7, name: 0x14, attr: ['BSETI'] },
   ]}

Encoding (RV64)

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x13, attr: ['OP-IMM'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['BSETI'] },
       { bits:  5, name: 'rs1' },
       { bits:  6, name: 'shamt' },
       { bits:  6, name: 0x0a, attr: ['BSETI'] },
   ]}

Description
   This instruction returns *rs1* with a single bit set at the index
   specified in *shamt*. The index is read from the lower log2(XLEN)
   bits of *shamt*. For RV32, the encodings corresponding to shamt[5]=1
   are reserved.

Operation

.. code:: sail

   let index = shamt & (XLEN - 1);
   X(rd) = X(rs1) | (1 << index)

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbs (`Single-bit                  | 0.93            | Frozen          |
| instructions <#zbs>`__)           |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-clmul:

clmul
-----

Synopsis
   Carry-less multiply (low-part)

Mnemonic
   clmul *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x33, attr: ['OP'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['CLMUL'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x5, attr: ['MINMAX/CLMUL'] },
   ]}

Description
   clmul produces the lower half of the 2·XLEN carry-less product.

Operation

.. code:: sail

   let rs1_val = X(rs1);
   let rs2_val = X(rs2);
   let output : xlenbits = 0;

   foreach (i from 0 to (xlen - 1) by 1) {
      output = if   ((rs2_val >> i) & 1)
               then output ^ (rs1_val << i);
               else output;
   }

   X[rd] = output

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbc (`Carry-less                  | 0.93            | Frozen          |
| multiplication <#zbc>`__)         |                 |                 |
+-----------------------------------+-----------------+-----------------+
| Zbkc (`Carry-less multiplication  | v0.9.4          | Frozen          |
| for Cryptography <#zbkc>`__)      |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-clmulh:

clmulh
------

Synopsis
   Carry-less multiply (high-part)

Mnemonic
   clmulh *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x33, attr: ['OP'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x3, attr: ['CLMULH'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x5, attr: ['MINMAX/CLMUL'] },
   ]}

Description
   clmulh produces the upper half of the 2·XLEN carry-less product.

Operation

.. code:: sail

   let rs1_val = X(rs1);
   let rs2_val = X(rs2);
   let output : xlenbits = 0;

   foreach (i from 1 to xlen by 1) {
      output = if   ((rs2_val >> i) & 1)
               then output ^ (rs1_val >> (xlen - i));
               else output;
   }

   X[rd] = output

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbc (`Carry-less                  | 0.93            | Frozen          |
| multiplication <#zbc>`__)         |                 |                 |
+-----------------------------------+-----------------+-----------------+
| Zbkc (`Carry-less multiplication  | v0.9.4          | Frozen          |
| for Cryptography <#zbkc>`__)      |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-clmulr:

clmulr
------

Synopsis
   Carry-less multiply (reversed)

Mnemonic
   clmulr *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x33, attr: ['OP'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x2, attr: ['CLMULR'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x5, attr: ['MINMAX/CLMUL'] },
   ]}

Description
   **clmulr** produces bits 2·XLEN−2:XLEN-1 of the 2·XLEN carry-less
   product.

Operation

.. code:: sail

   let rs1_val = X(rs1);
   let rs2_val = X(rs2);
   let output : xlenbits = 0;

   foreach (i from 0 to (xlen - 1) by 1) {
      output = if   ((rs2_val >> i) & 1)
               then output ^ (rs1_val >> (xlen - i - 1));
               else output;
   }

   X[rd] = output

.. note::

   The **clmulr** instruction is used to accelerate CRC calculations.
   The **r** in the instruction’s mnemonic stands for *reversed*, as the
   instruction is equivalent to bit-reversing the inputs, performing a
   **clmul**, then bit-reversing the output.

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbc (`Carry-less                  | 0.93            | Frozen          |
| multiplication <#zbc>`__)         |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-clz:

clz
---

Synopsis
   Count leading zero bits

Mnemonic
   clz *rd*, *rs*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x13, attr: ['OP-IMM'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['CLZ']  },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 0x0, attr: ['CLZ'] },
       { bits:  7, name: 0x30, attr: ['CLZ']  },
   ]}

Description
   This instruction counts the number of 0’s before the first 1,
   starting at the most-significant bit (i.e., XLEN-1) and progressing
   to bit 0. Accordingly, if the input is 0, the output is XLEN, and if
   the most-significant bit of the input is a 1, the output is 0.

Operation

.. code:: sail

   val HighestSetBit : forall ('N : Int), 'N >= 0. bits('N) -> int

   function HighestSetBit x = {
     foreach (i from (xlen - 1) to 0 by 1 in dec)
       if [x[i]] == 0b1 then return(i) else ();
     return -1;
   }

   let rs = X(rs);
   X[rd] = (xlen - 1) - HighestSetBit(rs);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-clzw:

clzw
----

Synopsis
   Count leading zero bits in word

Mnemonic
   clzw *rd*, *rs*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x1b, attr: ['OP-IMM-32'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['CLZW'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 0x0, attr: ['CLZW'] },
       { bits:  7, name: 0x30, attr: ['CLZW'] },
   ]}

Description
   This instruction counts the number of 0’s before the first 1 starting
   at bit 31 and progressing to bit 0. Accordingly, if the
   least-significant word is 0, the output is 32, and if the
   most-significant bit of the word (i.e., bit 31) is a 1, the output is
   0.

Operation

.. code:: sail

   val HighestSetBit32 : forall ('N : Int), 'N >= 0. bits('N) -> int

   function HighestSetBit32 x = {
     foreach (i from 31 to 0 by 1 in dec)
       if [x[i]] == 0b1 then return(i) else ();
     return -1;
   }

   let rs = X(rs);
   X[rd] = 31 - HighestSetBit(rs);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-cpop:

cpop
----

Synopsis
   Count set bits

Mnemonic
   cpop *rd*, *rs*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x13, attr: ['OP-IMM'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['CPOP'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 0x2, attr: ['CPOP'] },
       { bits:  7, name: 0x30, attr: ['CPOP'] },
   ]}

Description
   This instructions counts the number of 1’s (i.e., set bits) in the
   source register.

Operation

.. code:: sail

   let bitcount = 0;
   let rs = X(rs);

   foreach (i from 0 to (xlen - 1) in inc)
       if rs[i] == 0b1 then bitcount = bitcount + 1 else ();

   X[rd] = bitcount

.. note::

   This operations is known as population count, popcount, sideways sum,
   bit summation, or Hamming weight.

   The GCC builtin function ``__builtin_popcount (unsigned int x)`` is
   implemented by cpop on RV32 and by **cpopw** on RV64. The GCC builtin
   function ``__builtin_popcountl (unsigned long x)`` for LP64 is
   implemented by **cpop** on RV64.

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-cpopw:

cpopw
-----

Synopsis
   Count set bits in word

Mnemonic
   cpopw *rd*, *rs*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x1b, attr: ['OP-IMM-32'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['CPOPW'] },
       { bits:  5, name: 'rs' },
       { bits:  5, name: 0x2, attr: ['CPOPW'] },
       { bits:  7, name: 0x30, attr: ['CPOPW'] },
   ]}

Description
   This instructions counts the number of 1’s (i.e., set bits) in the
   least-significant word of the source register.

Operation

.. code:: sail

   let bitcount = 0;
   let val = X(rs);

   foreach (i from 0 to 31 in inc)
       if val[i] == 0b1 then bitcount = bitcount + 1 else ();

   X[rd] = bitcount

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-ctz:

ctz
---

Synopsis
   Count trailing zeros

Mnemonic
   ctz *rd*, *rs*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x13, attr: ['OP-IMM'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['CTZ/CTZW'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 0x1, attr: ['CTZ/CTZW'] },
       { bits:  7, name: 0x30, attr: ['CTZ/CTZW'] },
   ]}

Description
   This instruction counts the number of 0’s before the first 1,
   starting at the least-significant bit (i.e., 0) and progressing to
   the most-significant bit (i.e., XLEN-1). Accordingly, if the input is
   0, the output is XLEN, and if the least-significant bit of the input
   is a 1, the output is 0.

Operation

.. code:: sail

   val LowestSetBit : forall ('N : Int), 'N >= 0. bits('N) -> int

   function LowestSetBit x = {
     foreach (i from 0 to (xlen - 1) by 1 in dec)
       if [x[i]] == 0b1 then return(i) else ();
     return xlen;
   }

   let rs = X(rs);
   X[rd] = LowestSetBit(rs);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-ctzw:

ctzw
----

Synopsis
   Count trailing zero bits in word

Mnemonic
   ctzw *rd*, *rs*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x1b, attr: ['OP-IMM-32'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['CTZ/CTZW'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 0x1, attr: ['CTZ/CTZW'] },
       { bits:  7, name: 0x30, attr: ['CTZ/CTZW'] },
   ]}

Description
   This instruction counts the number of 0’s before the first 1,
   starting at the least-significant bit (i.e., 0) and progressing to
   the most-significant bit of the least-significant word (i.e., 31).
   Accordingly, if the least-significant word is 0, the output is 32,
   and if the least-significant bit of the input is a 1, the output is
   0.

Operation

.. code:: sail

   val LowestSetBit32 : forall ('N : Int), 'N >= 0. bits('N) -> int

   function LowestSetBit32 x = {
     foreach (i from 0 to 31 by 1 in dec)
       if [x[i]] == 0b1 then return(i) else ();
     return 32;
   }

   let rs = X(rs);
   X[rd] = LowestSetBit32(rs);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-max:

max
---

Synopsis
   Maximum

Mnemonic
   max *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x33, attr: ['OP'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x6, attr: ['MAX']},
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x05, attr: ['MINMAX/CLMUL'] },
   ]}

Description
   This instruction returns the larger of two signed integers.

Operation

.. code:: sail

   let rs1_val = X(rs1);
   let rs2_val = X(rs2);

   let result = if   rs1_val <_s rs2_val
                then rs2_val
                else rs1_val;

   X(rd) = result;

.. note::

   Calculating the absolute value of a signed integer can be performed
   using the following sequence: **neg rD,rS** followed by **max
   rD,rS,rD**. When using this common sequence, it is suggested that
   they are scheduled with no intervening instructions so that
   implementations that are so optimized can fuse them together.

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-maxu:

maxu
----

Synopsis
   Unsigned maximum

Mnemonic
   maxu *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x33, attr: ['OP'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x7, attr: ['MAXU']},
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x05, attr: ['MINMAX/CLMUL'] },
   ]}

Description
   This instruction returns the larger of two unsigned integers.

Operation

.. code:: sail

   let rs1_val = X(rs1);
   let rs2_val = X(rs2);

   let result = if   rs1_val <_u rs2_val
                then rs2_val
                else rs1_val;

   X(rd) = result;

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-min:

min
---

Synopsis
   Minimum

Mnemonic
   min *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x33, attr: ['OP'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x4, attr: ['MIN']},
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x05, attr: ['MINMAX/CLMUL'] },
   ]}

Description
   This instruction returns the smaller of two signed integers.

Operation

.. code:: sail

   let rs1_val = X(rs1);
   let rs2_val = X(rs2);

   let result = if   rs1_val <_s rs2_val
                then rs1_val
                else rs2_val;

   X(rd) = result;

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-minu:

minu
----

Synopsis
   Unsigned minimum

Mnemonic
   minu *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x33, attr: ['OP'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x5, attr: ['MINU']},
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x05, attr: ['MINMAX/CLMUL'] },
   ]}

Description
   This instruction returns the smaller of two unsigned integers.

Operation

.. code:: sail

   let rs1_val = X(rs1);
   let rs2_val = X(rs2);

   let result = if   rs1_val <_u rs2_val
                then rs1_val
                else rs2_val;

   X(rd) = result;

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-orc_b:

orc.b
-----

Synopsis
   Bitwise OR-Combine, byte granule

Mnemonic
   orc.b *rd*, *rs*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x13, attr: ['OP-IMM'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x5 },
       { bits:  5, name: 'rs' },
       { bits: 12, name: 0x287 }
   ]}

Description
   Combines the bits within each byte using bitwise logical OR. This
   sets the bits of each byte in the result *rd* to all zeros if no bit
   within the respective byte of *rs* is set, or to all ones if any bit
   within the respective byte of *rs* is set.

Operation

.. code:: sail

   let input = X(rs);
   let output : xlenbits = 0;

   foreach (i from 0 to (xlen - 8) by 8) {
      output[(i + 7)..i] = if   input[(i + 7)..i] == 0
                           then 0b00000000
                           else 0b11111111;
   }

   X[rd] = output;

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-orn:

orn
---

Synopsis
   OR with inverted operand

Mnemonic
   orn *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x33, attr: ['OP'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x6, attr: ['ORN']},
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x20, attr: ['ORN'] },
   ]}

Description
   This instruction performs the bitwise logical OR operation between
   *rs1* and the bitwise inversion of *rs2*.

Operation

.. code:: sail

   X(rd) = X(rs1) | ~X(rs2);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+
| Zbkb (`Bit-manipulation for       | v0.9.4          | Frozen          |
| Cryptography <#zbkb>`__)          |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-pack:

pack
----

Synopsis
   Pack the low halves of *rs1* and *rs2* into *rd*.

Mnemonic
   pack *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       {bits:  7, name: 0x33, attr: ['OP'] },
       {bits: 5, name: 'rd'},
       {bits: 3, name: 0x4, attr:['PACK']},
       {bits: 5, name: 'rs1'},
       {bits: 5, name: 'rs2'},
       {bits: 7, name: 0x4, attr:['PACK']},
   ]}

Description
   The pack instruction packs the XLEN/2-bit lower halves of *rs1* and
   *rs2* into *rd*, with *rs1* in the lower half and *rs2* in the upper
   half.

Operation

.. code:: sail

   let lo_half : bits(xlen/2) = X(rs1)[xlen/2-1..0];
   let hi_half : bits(xlen/2) = X(rs2)[xlen/2-1..0];
   X(rd) = EXTZ(hi_half @ lo_half);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbkb (`Bit-manipulation for       | v0.9.4          | Frozen          |
| Cryptography <#zbkb>`__)          |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-packh:

packh
-----

Synopsis
   Pack the low bytes of *rs1* and *rs2* into *rd*.

Mnemonic
   packh *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       {bits:  7, name: 0x33, attr: ['OP'] },
       {bits: 5, name: 'rd'},
       {bits: 3, name: 0x7, attr: ['PACKH']},
       {bits: 5, name: 'rs1'},
       {bits: 5, name: 'rs2'},
       {bits: 7, name: 0x4, attr: ['PACKH']},
   ]}

Description
   And the packh instruction packs the least-significant bytes of *rs1*
   and *rs2* into the 16 least-significant bits of *rd*, zero extending
   the rest of *rd*.

Operation

.. code:: sail

   let lo_half : bits(8) = X(rs1)[7..0];
   let hi_half : bits(8) = X(rs2)[7..0];
   X(rd) = EXTZ(hi_half @ lo_half);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbkb (`Bit-manipulation for       | v0.9.4          | Frozen          |
| Cryptography <#zbkb>`__)          |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-packw:

packw
-----

Synopsis
   Pack the low 16-bits of *rs1* and *rs2* into *rd* on RV64.

Mnemonic
   packw *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
   {bits: 2, name: 0x3},
   {bits: 5, name: 0xe},
   {bits: 5, name: 'rd'},
   {bits: 3, name: 0x4},
   {bits: 5, name: 'rs1'},
   {bits: 5, name: 'rs2'},
   {bits: 7, name: 0x4},
   ]}

Description
   This instruction packs the low 16 bits of *rs1* and *rs2* into the 32
   least-significant bits of *rd*, sign extending the 32-bit result to
   the rest of *rd*. This instruction only exists on RV64 based systems.

Operation

.. code:: sail

   let lo_half : bits(16) = X(rs1)[15..0];
   let hi_half : bits(16) = X(rs2)[15..0];
   X(rd) = EXTS(hi_half @ lo_half);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbkb (`Bit-manipulation for       | v0.9.4          | Frozen          |
| Cryptography <#zbkb>`__)          |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-rev8:

rev8
----

Synopsis
   Byte-reverse register

Mnemonic
   rev8 *rd*, *rs*

Encoding (RV32)

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x13, attr: ['OP-IMM'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x5 },
       { bits:  5, name: 'rs' },
       { bits: 12, name: 0x698 }
   ]}

Encoding (RV64)

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x13, attr: ['OP-IMM'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x5 },
       { bits:  5, name: 'rs' },
       { bits: 12, name: 0x6b8 }
   ]}

Description
   This instruction reverses the order of the bytes in *rs*.

Operation

.. code:: sail

   let input = X(rs);
   let output : xlenbits = 0;
   let j = xlen - 1;

   foreach (i from 0 to (xlen - 8) by 8) {
      output[i..(i + 7)] = input[(j - 7)..j];
      j = j - 8;
   }

   X[rd] = output

.. note::

   The **rev8** mnemonic corresponds to different instruction encodings
   in RV32 and RV64.

.. note::

   The byte-reverse operation is only available for the full register
   width. To emulate word-sized and halfword-sized byte-reversal,
   perform a ``rev8 rd,rs`` followed by a ``srai rd,rd,K``, where K is
   XLEN-32 and XLEN-16, respectively.

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+
| Zbkb (`Bit-manipulation for       | v0.9.4          | Frozen          |
| Cryptography <#zbkb>`__)          |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-revb:

rev.b
-----

Synopsis
   Reverse the bits in each byte of a source register.

Mnemonic
   rev.b *rd*, *rs*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x13, attr: ['OP-IMM'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x5 },
       { bits:  5, name: 'rs' },
       { bits: 12, name: 0x687 }
   ]}

Description
   This instruction reverses the order of the bits in every byte of a
   register.

Operation

.. code:: sail

   result : xlenbits = EXTZ(0b0);
   foreach (i from 0 to sizeof(xlen) by 8) {
       result[i+7..i] = reverse_bits_in_byte(X(rs1)[i+7..i]);
   };
   X(rd) = result;

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbkb (`Bit-manipulation for       | v0.9.4          | Frozen          |
| Cryptography <#zbkb>`__)          |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-rol:

rol
---

Synopsis
   Rotate Left (Register)

Mnemonic
   rol *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x33, attr: ['OP'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['ROL']},
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x30, attr: ['ROL'] },
   ]}

Description
   This instruction performs a rotate left of *rs1* by the amount in
   least-significant log2(XLEN) bits of *rs2*.

Operation

.. code:: sail

   let shamt = if   xlen == 32
               then X(rs2)[4..0]
               else X(rs2)[5..0];
   let result = (X(rs1) << shamt) | (X(rs1) >> (xlen - shamt));

   X(rd) = result;

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+
| Zbkb (`Bit-manipulation for       | v0.9.4          | Frozen          |
| Cryptography <#zbkb>`__)          |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-rolw:

rolw
----

Synopsis
   Rotate Left Word (Register)

Mnemonic
   rolw *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x3b, attr: ['OP-32'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['ROLW']},
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x30, attr: ['ROLW'] },
   ]}

Description
   This instruction performs a rotate left on the least-significant word
   of *rs1* by the amount in least-significant 5 bits of *rs2*. The
   resulting word value is sign-extended by copying bit 31 to all of the
   more-significant bits.

Operation

.. code:: sail

   let rs1 = EXTZ(X(rs1)[31..0])
   let shamt = X(rs2)[4..0];
   let result = (rs1 << shamt) | (rs1 >> (32 - shamt));
   X(rd) = EXTS(result[31..0]);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+
| Zbkb (`Bit-manipulation for       | v0.9.4          | Frozen          |
| Cryptography <#zbkb>`__)          |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-ror:

ror
---

Synopsis
   Rotate Right

Mnemonic
   ror *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x33, attr: ['OP'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x5, attr: ['ROR']},
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x30, attr: ['ROR'] },
   ]}

Description
   This instruction performs a rotate right of *rs1* by the amount in
   least-significant log2(XLEN) bits of *rs2*.

Operation

.. code:: sail

   let shamt = if   xlen == 32
               then X(rs2)[4..0]
               else X(rs2)[5..0];
   let result = (X(rs1) >> shamt) | (X(rs1) << (xlen - shamt));

   X(rd) = result;

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+
| Zbkb (`Bit-manipulation for       | v0.9.4          | Frozen          |
| Cryptography <#zbkb>`__)          |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-rori:

rori
----

Synopsis
   Rotate Right (Immediate)

Mnemonic
   rori *rd*, *rs1*, *shamt*

Encoding (RV32)

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x13, attr: ['OP-IMM'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x5, attr: ['RORI']},
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'shamt' },
       { bits:  7, name: 0x30, attr: ['RORI'] },
   ]}

Encoding (RV64)

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x13, attr: ['OP-IMM'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x5, attr: ['RORI']},
       { bits:  5, name: 'rs1' },
       { bits:  6, name: 'shamt' },
       { bits:  6, name: 0x18, attr: ['RORI'] },
   ]}

Description
   This instruction performs a rotate right of *rs1* by the amount in
   the least-significant log2(XLEN) bits of *shamt*. For RV32, the
   encodings corresponding to shamt[5]=1 are reserved.

Operation

.. code:: sail

   let shamt = if   xlen == 32
               then shamt[4..0]
               else shamt[5..0];
   let result = (X(rs1) >> shamt) | (X(rs1) << (xlen - shamt));

   X(rd) = result;

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+
| Zbkb (`Bit-manipulation for       | v0.9.4          | Frozen          |
| Cryptography <#zbkb>`__)          |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-roriw:

roriw
-----

Synopsis
   Rotate Right Word by Immediate

Mnemonic
   roriw *rd*, *rs1*, *shamt*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x1b, attr: ['OP-IMM-32'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x5, attr: ['RORIW']},
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'shamt' },
       { bits:  7, name: 0x30, attr: ['RORIW'] },
   ]}

Description
   This instruction performs a rotate right on the least-significant
   word of *rs1* by the amount in the least-significant log2(XLEN) bits
   of *shamt*. The resulting word value is sign-extended by copying bit
   31 to all of the more-significant bits.

Operation

.. code:: sail

   let rs1_data = EXTZ(X(rs1)[31..0];
   let result = (rs1_data >> shamt) | (rs1_data << (32 - shamt));
   X(rd) = EXTS(result[31..0]);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+
| Zbkb (`Bit-manipulation for       | v0.9.4          | Frozen          |
| Cryptography <#zbkb>`__)          |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-rorw:

rorw
----

Synopsis
   Rotate Right Word (Register)

Mnemonic
   rorw *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x3b, attr: ['OP-32'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x5, attr: ['RORW']},
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x30, attr: ['RORW'] },
   ]}

Description
   This instruction performs a rotate right on the least-significant
   word of *rs1* by the amount in least-significant 5 bits of *rs2*. The
   resultant word is sign-extended by copying bit 31 to all of the
   more-significant bits.

Operation

.. code:: sail

   let rs1 = EXTZ(X(rs1)[31..0])
   let shamt = X(rs2)[4..0];
   let result = (rs1 >> shamt) | (rs1 << (32 - shamt));
   X(rd) = EXTS(result);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+
| Zbkb (`Bit-manipulation for       | v0.9.4          | Frozen          |
| Cryptography <#zbkb>`__)          |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-sext_b:

sext.b
------

Synopsis
   Sign-extend byte

Mnemonic
   sext.b *rd*, *rs*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x13, attr: ['OP-IMM'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['SEXT.B/SEXT.H'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 0x04, attr: ['SEXT.B'] },
       { bits:  7, name: 0x30 },
   ]}

Description
   This instruction sign-extends the least-significant byte in the
   source to XLEN by copying the most-significant bit in the byte (i.e.,
   bit 7) to all of the more-significant bits.

Operation

.. code:: sail

   X(rd) = EXTS(X(rs)[7..0]);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-sext_h:

sext.h
------

Synopsis
   Sign-extend halfword

Mnemonic
   sext.h *rd*, *rs*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x13, attr: ['OP-IMM'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['SEXT.B/SEXT.H'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 0x05, attr: ['SEXT.H'] },
       { bits:  7, name: 0x30 },
   ]}

Description
   This instruction sign-extends the least-significant halfword in *rs*
   to XLEN by copying the most-significant bit in the halfword (i.e.,
   bit 15) to all of the more-significant bits.

Operation

.. code:: sail

   X(rd) = EXTS(X(rs)[15..0]);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-sh1add:

sh1add
------

Synopsis
   Shift left by 1 and add

Mnemonic
   sh1add *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x33, attr: ['OP'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x2, attr: ['SH1ADD'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x10, attr: ['SH1ADD'] },
   ]}

Description
   This instruction shifts *rs1* to the left by 1 bit and adds it to
   *rs2*.

Operation

.. code:: sail

   X(rd) = X(rs2) + (X(rs1) << 1);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zba (`Address generation          | 0.93            | Frozen          |
| instructions <#zba>`__)           |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-sh1add_uw:

sh1add.uw
---------

Synopsis
   Shift unsigned word left by 1 and add

Mnemonic
   sh1add.uw *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x3b, attr: ['OP-32'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x2, attr: ['SH1ADD.UW'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x10, attr: ['SH1ADD.UW']  },
   ]}

Description
   This instruction performs an XLEN-wide addition of two addends. The
   first addend is *rs2*. The second addend is the unsigned value formed
   by extracting the least-significant word of *rs1* and shifting it
   left by 1 place.

Operation

.. code:: sail

   let base = X(rs2);
   let index = EXTZ(X(rs1)[31..0]);

   X(rd) = base + (index << 1);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zba (`Address generation          | 0.93            | Frozen          |
| instructions <#zba>`__)           |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-sh2add:

sh2add
------

Synopsis
   Shift left by 2 and add

Mnemonic
   sh2add *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x33, attr: ['OP'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x4, attr: ['SH2ADD'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x10, attr: ['SH2ADD'] },
   ]}

Description
   This instruction shifts *rs1* to the left by 2 places and adds it to
   *rs2*.

Operation

.. code:: sail

   X(rd) = X(rs2) + (X(rs1) << 2);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zba (`Address generation          | 0.93            | Frozen          |
| instructions <#zba>`__)           |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-sh2add_uw:

sh2add.uw
---------

Synopsis
   Shift unsigned word left by 2 and add

Mnemonic
   sh2add.uw *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x3b, attr: ['OP-32'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x4, attr: ['SH2ADD.UW'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x10, attr: ['SH2ADD.UW'] },
   ]}

Description
   This instruction performs an XLEN-wide addition of two addends. The
   first addend is *rs2*. The second addend is the unsigned value formed
   by extracting the least-significant word of *rs1* and shifting it
   left by 2 places.

Operation

.. code:: sail

   let base = X(rs2);
   let index = EXTZ(X(rs1)[31..0]);

   X(rd) = base + (index << 2);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zba (`Address generation          | 0.93            | Frozen          |
| instructions <#zba>`__)           |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-sh3add:

sh3add
------

Synopsis
   Shift left by 3 and add

Mnemonic
   sh3add *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x33, attr: ['OP'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x6, attr: ['SH3ADD'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x10, attr: ['SH3ADD'] },
   ]}

Description
   This instruction shifts *rs1* to the left by 3 places and adds it to
   *rs2*.

Operation

.. code:: sail

   X(rd) = X(rs2) + (X(rs1) << 3);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zba (`Address generation          | 0.93            | Frozen          |
| instructions <#zba>`__)           |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-sh3add_uw:

sh3add.uw
---------

Synopsis
   Shift unsigned word left by 3 and add

Mnemonic
   sh3add.uw *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x3b, attr: ['OP-32'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x6, attr: ['SH3ADD.UW'] },
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x10, attr: ['SH3ADD.UW'] },
   ]}

Description
   This instruction performs an XLEN-wide addition of two addends. The
   first addend is *rs2*. The second addend is the unsigned value formed
   by extracting the least-significant word of *rs1* and shifting it
   left by 3 places.

Operation

.. code:: sail

   let base = X(rs2);
   let index = EXTZ(X(rs1)[31..0]);

   X(rd) = base + (index << 3);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zba (`Address generation          | 0.93            | Frozen          |
| instructions <#zba>`__)           |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-slli_uw:

slli.uw
-------

Synopsis
   Shift-left unsigned word (Immediate)

Mnemonic
   slli.uw *rd*, *rs1*, *shamt*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x1b, attr: ['OP-IMM-32'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x1, attr: ['SLLI.UW'] },
       { bits:  5, name: 'rs1' },
       { bits:  6, name: 'shamt' },
       { bits:  6, name: 0x02, attr: ['SLLI.UW'] },
   ]}

Description
   This instruction takes the least-significant word of *rs1*,
   zero-extends it, and shifts it left by the immediate.

Operation

.. code:: sail

   X(rd) = (EXTZ(X(rs)[31..0]) << shamt);

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zba (`Address generation          | 0.93            | Frozen          |
| instructions <#zba>`__)           |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. note::

   This instruction is the same as **slli** with **zext.w** performed on
   *rs1* before shifting.

.. _insns-unzip:

unzip
-----

Synopsis
   Implements the inverse of the zip instruction.

Mnemonic
   unzip *rd*, *rs*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
   {bits: 2, name: 0x3},
   {bits: 5, name: 0x4},
   {bits: 5, name: 'rd'},
   {bits: 3, name: 0x5},
   {bits: 5, name: 'rs1'},
   {bits: 5, name: 0x1f},
   {bits: 7, name: 0x4},
   ]}

Description
   This instruction gathers bits from the high and low halves of the
   source word into odd/even bit positions in the destination word. It
   is the inverse of the `zip <#insns-zip>`__ instruction. This
   instruction is available only on RV32.

Operation

.. code:: sail

   foreach (i from 0 to xlen/2-1) {
     X(rd)[i] = X(rs1)[2*i]
     X(rd)[i+xlen/2] = X(rs1)[2*i+1]
   }

.. note::

   This instruction is useful for implementing the SHA3 cryptographic
   hash function on a 32-bit architecture, as it implements the
   bit-interleaving operation used to speed up the 64-bit rotations
   directly.

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbkb (`Bit-manipulation for       | v0.9.4          | Frozen          |
| Cryptography <#zbkb>`__) (RV32)   |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-xnor:

xnor
----

Synopsis
   Exclusive NOR

Mnemonic
   xnor *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x33, attr: ['OP'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x4, attr: ['XNOR']},
       { bits:  5, name: 'rs1' },
       { bits:  5, name: 'rs2' },
       { bits:  7, name: 0x20, attr: ['XNOR'] },
   ]}

Description
   This instruction performs the bit-wise exclusive-NOR operation on
   *rs1* and *rs2*.

Operation

.. code:: sail

   X(rd) = ~(X(rs1) ^ X(rs2));

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+
| Zbkb (`Bit-manipulation for       | v0.9.4          | Frozen          |
| Cryptography <#zbkb>`__)          |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-xpermb:

xperm.b
-------

Synopsis
   Byte-wise lookup of indices into a vector in registers.

Mnemonic
   xperm.b *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
   {bits: 2, name: 0x3},
   {bits: 5, name: 0xc},
   {bits: 5, name: 'rd'},
   {bits: 3, name: 0x4},
   {bits: 5, name: 'rs1'},
   {bits: 5, name: 'rs2'},
   {bits: 7, name: 0x14},
   ]}

Description
   The xperm.b instruction operates on bytes. The *rs1* register
   contains a vector of XLEN/8 8-bit elements. The *rs2* register
   contains a vector of XLEN/8 8-bit indexes. The result is each element
   in *rs2* replaced by the indexed element in *rs1*, or zero if the
   index into *rs2* is out of bounds.

Operation

.. code:: sail

   val xpermb_lookup : (bits(8), xlenbits) -> bits(8)
   function xpermb_lookup (idx, lut) = {
       (lut >> (idx @ 0b000))[7..0]
   }

   function clause execute ( XPERM_B (rs2,rs1,rd)) = {
       result : xlenbits = EXTZ(0b0);
       foreach(i from 0 to xlen by 8) {
           result[i+7..i] = xpermn_lookup(X(rs2)[i+7..i], X(rs1));
       };
       X(rd) = result;
       RETIRE_SUCCESS
   }

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbkx (`Crossbar                   | v0.9.4          | Frozen          |
| permutations <#zbkx>`__)          |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-xpermn:

xperm.n
-------

Synopsis
   Nibble-wise lookup of indices into a vector.

Mnemonic
   xperm.n *rd*, *rs1*, *rs2*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
   {bits: 2, name: 0x3},
   {bits: 5, name: 0xc},
   {bits: 5, name: 'rd'},
   {bits: 3, name: 0x2},
   {bits: 5, name: 'rs1'},
   {bits: 5, name: 'rs2'},
   {bits: 7, name: 0x14},
   ]}

Description
   The xperm.n instruction operates on nibbles. The *rs1* register
   contains a vector of XLEN/4 4-bit elements. The *rs2* register
   contains a vector of XLEN/4 4-bit indexes. The result is each element
   in *rs2* replaced by the indexed element in *rs1*, or zero if the
   index into *rs2* is out of bounds.

Operation

.. code:: sail

   val xpermn_lookup : (bits(4), xlenbits) -> bits(4)
   function xpermn_lookup (idx, lut) = {
       (lut >> (idx @ 0b00))[3..0]
   }

   function clause execute ( XPERM_N (rs2,rs1,rd)) = {
       result : xlenbits = EXTZ(0b0);
       foreach(i from 0 to xlen by 4) {
           result[i+3..i] = xpermn_lookup(X(rs2)[i+3..i], X(rs1));
       };
       X(rd) = result;
       RETIRE_SUCCESS
   }

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbkx (`Crossbar                   | v0.9.4          | Frozen          |
| permutations <#zbkx>`__)          |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-zext_h:

zext.h
------

Synopsis
   Zero-extend halfword

Mnemonic
   zext.h *rd*, *rs*

Encoding (RV32)

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x33, attr: ['OP'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x4, attr: ['ZEXT.H']},
       { bits:  5, name: 'rs' },
       { bits:  5, name: 0x00 },
       { bits:  7, name: 0x04 },
   ]}

Encoding (RV64)

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
       { bits:  7, name: 0x3b, attr: ['OP-32'] },
       { bits:  5, name: 'rd' },
       { bits:  3, name: 0x4, attr: ['ZEXT.H']},
       { bits:  5, name: 'rs' },
       { bits:  5, name: 0x00 },
       { bits:  7, name: 0x04 },
   ]}

Description
   This instruction zero-extends the least-significant halfword of the
   source to XLEN by inserting 0’s into all of the bits more significant
   than 15.

Operation

.. code:: sail

   X(rd) = EXTZ(X(rs)[15..0]);

.. note::

   The **zext.h** mnemonic corresponds to different instruction
   encodings in RV32 and RV64.

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbb (`Basic                       | 0.93            | Frozen          |
| bit-manipulation <#zbb>`__)       |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. _insns-zip:

zip
---

Synopsis
   Gather odd and even bits of the source word into upper/lower halves
   of the destination.

Mnemonic
   zip *rd*, *rs*

Encoding

::

   Failed to generate image: no implicit conversion of nil into String
   {reg:[
   {bits: 2, name: 0x3},
   {bits: 5, name: 0x4},
   {bits: 5, name: 'rd'},
   {bits: 3, name: 0x1},
   {bits: 5, name: 'rs1'},
   {bits: 5, name: 0x1e},
   {bits: 7, name: 0x4},
   ]}

Description
   This instruction scatters all of the odd and even bits of a source
   word into the high and low halves of a destination word. It is the
   inverse of the `unzip <#insns-unzip>`__ instruction. This instruction
   is available only on RV32.

Operation

.. code:: sail

   foreach (i from 0 to xlen/2-1) {
     X(rd)[2*i] = X(rs1)[i]
     X(rd)[2*i+1] = X(rs1)[i+xlen/2]
   }

.. note::

   This instruction is useful for implementing the SHA3 cryptographic
   hash function on a 32-bit architecture, as it implements the
   bit-interleaving operation used to speed up the 64-bit rotations
   directly.

Included in

+-----------------------------------+-----------------+-----------------+
| Extension                         | Minimum version | Lifecycle state |
+===================================+=================+=================+
| Zbkb (`Bit-manipulation for       | v0.9.4          | Frozen          |
| Cryptography <#zbkb>`__) (RV32)   |                 |                 |
+-----------------------------------+-----------------+-----------------+

.. __software_optimization_guide:

Software optimization guide
===========================

.. __strlen:

strlen
------

The **orc.b** instruction allows for the efficient detection of **NUL**
bytes in an XLEN-sized chunk of data:

-  the result of **orc.b** on a chunk that does not contain any **NUL**
   bytes will be all-ones, and

-  after a bitwise-negation of the result of **orc.b**, the number of
   data bytes before the first **NUL** byte (if any) can be detected by
   **ctz**/**clz** (depending on the endianness of data).

A full example of a **strlen** function, which uses these techniques and
also demonstrates the use of it for unaligned/partial data, is the
following:

.. code:: asm

   #include <sys/asm.h>

       .text
       .globl strlen
       .type  strlen, @function
   strlen:
       andi    a3, a0, (SZREG-1)   // offset
       andi    a1, a0, -SZREG      // align pointer
   .Lprologue:
       li      a4, SZREG
       sub     a4, a4, a3          // XLEN - offset
       slli    a3, a3, PTRLOG      // offset * 8
       REG_L   a2, 0(a1)           // chunk
       /*
        * Shift the partial/unaligned chunk we loaded to remove the bytes
        * from before the start of the string, adding NUL bytes at the end.
        */
   #if __BYTE_ORDER__ == __ORDER_LITTLE_ENDIAN__
       srl a2, a2 ,a3          // chunk >> (offset * 8)
   #else
       sll     a2, a2, a3
   #endif
       orc.b   a2, a2
       not a2, a2
       /*
        * Non-NUL bytes in the string have been expanded to 0x00, while
        * NUL bytes have become 0xff.  Search for the first set bit
        * (corresponding to a NUL byte in the original chunk).
        */
   #if __BYTE_ORDER__ == __ORDER_LITTLE_ENDIAN__
       ctz     a2, a2
   #else
       clz     a2, a2
   #endif
       /*
        * The first chunk is special: compare against the number of valid
        * bytes in this chunk.
        */
       srli    a0, a2, 3
       bgtu    a4, a0, .Ldone
       addi    a3, a1, SZREG
       li      a4, -1
       .align 2
       /*
        * Our critical loop is 4 instructions and processes data in 4 byte
        * or 8 byte chunks.
        */
   .Lloop:
       REG_L   a2, SZREG(a1)
       addi    a1, a1, SZREG
       orc.b   a2, a2
       beq     a2, a4, .Lloop

   .Lepilogue:
       not     a2, a2
   #if __BYTE_ORDER__ == __ORDER_LITTLE_ENDIAN__
       ctz     a2, a2
   #else
       clz     a2, a2
   #endif
       sub     a1, a1, a3
       add a0, a0, a1
       srli    a2, a2, 3
       add     a0, a0, a2
   .Ldone:
       ret

.. __strcmp:

strcmp
------

.. code:: asm

   #include <sys/asm.h>

     .text
     .globl strcmp
     .type  strcmp, @function
   strcmp:
     or    a4, a0, a1
     li    t2, -1
     and   a4, a4, SZREG-1
     bnez  a4, .Lsimpleloop

     # Main loop for aligned strings
   .Lloop:
     REG_L a2, 0(a0)
     REG_L a3, 0(a1)
     orc.b t0, a2
     bne   t0, t2, .Lfoundnull
     addi  a0, a0, SZREG
     addi  a1, a1, SZREG
     beq   a2, a3, .Lloop

     # Words don't match, and no null byte in first word.
     # Get bytes in big-endian order and compare.
   #if __BYTE_ORDER__ == __ORDER_LITTLE_ENDIAN__
     rev8  a2, a2
     rev8  a3, a3
   #endif
     # Synthesize (a2 >= a3) ? 1 : -1 in a branchless sequence.
     sltu a0, a2, a3
     neg  a0, a0
     ori  a0, a0, 1
     ret

   .Lfoundnull:
     # Found a null byte.
     # If words don't match, fall back to simple loop.
     bne   a2, a3, .Lsimpleloop

     # Otherwise, strings are equal.
     li    a0, 0
     ret

     # Simple loop for misaligned strings
   .Lsimpleloop:
     lbu   a2, 0(a0)
     lbu   a3, 0(a1)
     addi  a0, a0, 1
     addi  a1, a1, 1
     bne   a2, a3, 1f
     bnez  a2, .Lsimpleloop

   1:
     sub   a0, a2, a3
     ret

   .size   strcmp, .-strcmp
