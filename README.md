# 波函数塌陷
这个程序可以生成和输入图像局部相似的图像

<p align="center"><img alt="main collage" src="http://i.imgur.com/g1yGvL7.png"></p>
<p align="center"><img alt="main gif" src="http://i.imgur.com/sNuBVSr.gif"></p>

局部相似意味着：
 
* (C1) 输出图像中，每个NxN范围的像素图案，都至少在输入图像中出现一次。
* (Weak C2) 输入图像中各个像素图案的分布方式，应该和输出图像中的大量分布保持相识。换句话说，输出图像中某个图案的出现概率应该和输入图像一样。

In the examples typical value of N is 3.
在范例中，N一般是3。

<p align="center"><img alt="local similarity" src="http://i.imgur.com/KULGX86.png"></p>
 
WFC(波函数塌陷，下同) 以一种完全的“未被观察”的状态初始化输出图象，每个像素的值是输入图像的多种颜色的叠加（因此，如果输入的是黑白图像，那么“未被观察”的状态就是灰度中的某一级）。这种叠加态的系数都是实数，而不是复数，所以它不包含真的量子力学，但是这个算法受其启发。接下来，程序进入了 “观测-传播” 的循环：

* 每一个“观察”步骤，我们从未被观察的区域中选择一个NxN的区域，要保证这个区域有最小的香农熵(Shannon entropy).这个区域的状态根据它的系数和输入图像中各个图案的分布概率因此塌陷成一个有限的状态。

 
* 每个一个“传播”的步骤，上一个观察步骤得到的新的信息，将会在输出图像中传播。

 
每经过一个步骤，总体的熵将会减少，最后我们会有一个完全观察过的状态，波函数就塌陷。

 
有可能发生的情况是，在传播的过程中，某个像素的所有系数可能都会变成0。这意味着算法走进了一个死路。要想判定一个输入图像是否能够产生与其完全不同的图像，是一个NP困难的问题，所以不可能找到一个快速的，在有限的时间内完成。但是在实践中，这个算法出人意料的极少陷入自我矛盾的情况。
 
请观看一个视频讲解YouTube: [https://youtu.be/DOQTr2Xmlz0](https://youtu.be/DOQTr2Xmlz0)
 
## 算法


1. 读取输入图像然后对图案进行计数。
    1. (可选) 用旋转和翻转来增强图案数据。

2. 根据输出图像的大小创建一个数组（被称为“波”）。每个元素代表输出图像中一个NxN区域的状态。这个状态是每个输入图像NxN图案的bool系数（所以输出图像中每个像素的状态就是输入图像的各个颜色与实数系数的叠加）。
系数为假(false)时，表示对应的图案不能放置在这里，系数为真，表示对应的图案尚有可能出现在这里。

3.初始化“波”数组为，完全未被观察的状态，也就是所有的bool值为真。


4. 重复一下步骤:
    1. 观察:
        1. 寻找一个具有最小非零熵值的“波”元素。如果没有这样的元素 (就是所有的元素都只有0或未定义的熵) 那么就退出循环(4)并到步骤（5）。
        2. 利用这个元素的系数和图案的分布概率，把这个元素坍缩成有限状态.
    2. 传播: 把观察步骤里的产生的新的信息，传播到周围的元素。
	

5. 现在，所有的“波”数组的元素，要么在一个完全的被观测的状态（只有一个系数是真），或者出一个自相矛盾的状态（所有的系数都是假）。第一种情况，返回结果，第二种情况，完成程序但是不返回结果。

## Tilemap generation
The simplest nontrivial case of the algorithm is when NxN=1x2 (well, NxM). If we simplify it even further by storing not the probabilities of pairs of colors but the probabilities of colors themselves, we get what we call a "simple tiled model". The propagation phase in this model is just adjacency constraint propagation. It's convenient to initialize the simple tiled model with a list of tiles and their adjacency data (adjacency data can be viewed as a large set of very small samples) rather than a sample bitmap.

## 地砖生成
这个算法所能产生的最简单不平凡解的情况是当NxN为1x2时。如果我们把它进一步简单化为不存颜色对的概率而是颜色值本身概率，我们就得到一个“简单地砖模型”。这个模型里的传播步骤仅仅是近邻限制的传播。可以很方便的把这个模型初始化为一个砖块的列表以及他们的相邻性数据（相邻性数据可以被看成是很多小样本的大集合）而不是一个简单的位图。
The simplest nontrivial case of the algorithm is when NxN=1x2 (well, NxM). If we simplify it even further by storing not the probabilities of pairs of colors but the probabilities of colors themselves, we get what we call a "simple tiled model". The propagation phase in this model is just adjacency constraint propagation. It's convenient to initialize the simple tiled model with a list of tiles and their adjacency data (adjacency data can be viewed as a large set of very small samples) rather than a sample bitmap.


<p align="center">
  <a href="http://i.imgur.com/jIctSoT.gif">GIF</a> |
  <a href="http://i.imgur.com/jIctSoT.gifv">GIFV</a>
</p>

Lists of all the possible pairs of adjacent tiles in practical tilesets can be quite long, so I implemented a symmetry system for tiles to shorten the enumeration. In this system each tile should be assigned with its symmetry type.

在实际的地砖集合中，列出所有的可能的相邻的砖块的列表可能会非常长，所以我们构建了一个对称系统来减少枚举种类。这个系统中，每个地砖都有一个对应的枚举值。

<p align="center"><img alt="symmetries" src="http://i.imgur.com/9H0frmK.png"></p>

Note that the tiles have the same symmetry type as their assigned letters (or, in other words, actions of the 
dihedral group D4 are isomorphic for tiles and their corresponding letters). With this system it's enough to enumerate pairs of adjacent tiles only up to symmetry, which makes lists of adjacencies for tilesets with many symmetrical tiles (even the summer tileset, despite drawings not being symmetrical the system considers such tiles to be symmetrical) several times shorter.

要注意的是，这些砖块都有和他们被赋予的字母相同的对称类型（换句话说，二面体组D4的动作对于砖块及其各自的字母是同构的）。这个系统中，仅用对称度来给各种砖块赋予枚举值是足够用的，这样使得枚举列表短了好几倍（有些砖块并不对称但是系统依旧把他们当作对称的砖块）。

<p align="center">
	<img alt="knots" src="http://i.imgur.com/EnBkcVN.png">
	<img alt="tiled rooms" src="http://i.imgur.com/BruxOx9.png">
	<img alt="circuit 1" src="http://i.imgur.com/BYt7AR6.png">
	<img alt="circuit 2" src="http://i.imgur.com/yYHbMx8.png">
	<img alt="circles" src="http://i.imgur.com/Hrs0Ir8.png">
	<img alt="castle" src="http://i.imgur.com/Nd2mQOC.png">
	<img alt="summer 1" src="http://i.imgur.com/re8WBud.png">
	<img alt="summer 2" src="http://i.imgur.com/OmUHk1t.png">
</p>

Note that the unrestrained knot tileset (with all 5 tiles being allowed) is not interesting for WFC, because you can't run into a situation where you can't place a tile. We call tilesets with this property "easy". For example, Wang tilesets are easy. Without special heuristics easy tilesets don't produce interesting global arrangements, because correlations of tiles in easy tilesets quickly fall off with a distance.

注意，没有边缘限制的孤立砖块，对于WFC来说没有太大的兴趣，因为你总是能把他们摆下。这种图集对于WFC来说是“简单”的。王氏砖块是“简单”的。没有特殊的启发，简单的图集不会产生有趣的全局图案，因为砖块的相关性随着距离快速的消失。

Many interesting Wang tilesets can be found on [cr31's site](http://s358455341.websitehome.co.uk/stagecast/wang/tiles_e.html). Consider the "Dual" 2-edge tileset there. How can it generate knots (without t-junctions, not easy) while being easy? The answer is, it can only generate a narrow class of knots, it can't produce an arbitrary knot.

很多有趣的王氏砖块可以在这个地址找到[cr31's site](http://s358455341.websitehome.co.uk/stagecast/wang/tiles_e.html)。


## 更高的维度
WFC 在更高的维度与二维的情况完全一样, 尽管运行性能可能是个问题。这些体素模型是在N=2的情况下，使用5x5x5和5x5x2的砖块模型生成的（还考虑了高度，密度，曲率等）。


<p align="center"><img alt="voxels" src="http://i.imgur.com/hsqPdQl.png"></p>

高清图片链接: [1](http://i.imgur.com/0bsjlBY.png), [2](http://i.imgur.com/GduN0Vr.png), [3](http://i.imgur.com/IEOsbIy.png).


WFC生成体素模型和其他相关算法会在另一个单独的repo里。 

## Constrained synthesis
WFC algorithm supports constraints. Therefore, it can be easely combined with other generative algorithms or with manual creation.

## 有限制的生成算法
WFC 算法支持. 因此, 它可以和其他的生成算法向结合。

这是一个WFC自动完成一个由人类开始的关卡：

<p align="center">
  <a href="http://i.imgur.com/X3aNDUv.gif">GIF</a> |
  <a href="http://i.imgur.com/X3aNDUv.gifv">GIFV</a>
</p>

[ConvChain](https://github.com/mxgmn/ConvChain) algorithm satisfies the strong version of the condition (C2): the limit distribution of NxN patterns in the outputs it is producing is exactly the same as the distributions of patterns in the input. However, ConvChain doesn't satisfy (C1): it often produces noticeable artefacts. It makes sense to run ConvChain first to get a well-sampled configuration and then run WFC to correct local artefacts. This is similar to a common strategy in optimization: first run a Monte-Carlo method to find a point close to a global optimum and then run a gradient descent from that point for greater accuracy.

P. F. Harrison's [texture synthesis](https://github.com/mxgmn/SynTex) algorithm is significantly faster than WFC, but it has trouble with long correlations (for example, it's difficult for this algorithm to synthesize brick wall textures with correctly aligned bricks). But this is exactly where WFC shines, and Harrison's algorithm supports constraints. It makes sense first to generate a perfect brick wall blueprint with WFC and then run a constrained texture synthesis algorithm on that blueprint.

## Comments
Why the minimal entropy heuristic? I noticed that when humans draw something they often follow the minimal entropy heuristic themselves. That's why the algorithm is so enjoyable to watch.

The overlapping model relates to the simple tiled model the same way higher order Markov chains relate to order one Markov chains.

Note that the entropy of any node can't increase during the propagation phase, i.e. possibilities are not arising, but can be canceled. When propagation step can not decrease entropy further, we activate observation step. If the observation step can not decrease entropy, that means that the algorithm has finished working.

WFC's propagation phase is very similar to the loopy belief propagation algorithm. In fact, I first programmed belief propagation, but then switched to constraint propagation with a saved stationary distribution, because BP is significantly slower without a massive parallelization (on a CPU) and didn't produce significantly better results in my problems.

Note that the "Simple Knot" and "Trick Knot" samples have 3 colors, not 2.

One of the dimensions can be time. In particular, d-dimensional WFC captures the behaviour of any (d-1)-dimensional cellular automata.

## References
This project builds upon Paul Merrell's work on model synthesis, in particular discrete model synthesis chapter of [his dissertation](http://graphics.stanford.edu/~pmerrell/thesis.pdf). Paul propagates adjacency constraints in what we call a simple tiled model with a heuristic that tries to complete propagation in a small moving region.

It was also heavily influenced by declarative texture synthesis chapter of [Paul F. Harrison's dissertation](http://logarithmic.net/pfh-files/thesis/dissertation.pdf). Paul defines adjacency data of tiles by labeling their borders and uses backtracking search to fill the tilemap.

## Notable ports, forks and spinoffs

* Emil Ernerfeldt made a [C++ port](https://github.com/emilk/wfc).
* [Max Aller](https://github.com/nanodeath) made a Kotlin (JVM) library, [Kollapse](https://gitlab.com/nanodeath/kollapse).
* [Kevin Chapelier](https://github.com/kchapelier) made a [JavaScript port](http://www.kchapelier.com/wfc-example/overlapping-model.html).
* Oskar Stalberg programmed a 3d tiled model, a 2d tiled model for irregular grids on a sphere and is building beautiful 3d tilesets for them: [1](https://twitter.com/OskSta/status/787319655648100352), [2](https://twitter.com/OskSta/status/784847588893814785), [3](https://twitter.com/OskSta/status/784847933686575104), [4](https://twitter.com/OskSta/status/784848286272327680), [5](https://twitter.com/OskSta/status/793545297376972801), [6](https://twitter.com/OskSta/status/793806535898136576), [7](https://twitter.com/OskSta/status/802496920790777856), [8](https://twitter.com/OskSta/status/804291629561577472), [9](https://twitter.com/OskSta/status/806856212260278272), [10](https://twitter.com/OskSta/status/806904557502464000), [11](https://twitter.com/OskSta/status/818857408848130048), [12](https://twitter.com/OskSta/status/832633189277409280), [13](https://twitter.com/OskSta/status/851170356530475008), [14](https://twitter.com/OskSta/status/858301207936458752), [15](https://twitter.com/OskSta/status/863019585162932224).
* [Joseph Parker](https://github.com/selfsame) adapted [WFC to Unity](https://selfsame.itch.io/unitywfc) and used it generate skateparks in the [Proc Skater 2016](https://arcadia-clojure.itch.io/proc-skater-2016) game and [fantastic plateaus](https://twitter.com/jplur_/status/929482200034226176) in the 2017 game [Swapland](https://arcadia-clojure.itch.io/swapland).
* [Martin O'Leary](https://github.com/mewo2) applied a [WFC-like algorithm](https://github.com/mewo2/oisin) to poetry generation: [1](https://twitter.com/mewo2/status/789167437518217216), [2](https://twitter.com/mewo2/status/789177702620114945), [3](https://twitter.com/mewo2/status/789187174683987968), [4](https://twitter.com/mewo2/status/789897712372183041).
* [Nick Nenov](https://github.com/NNNenov) made a [3d voxel tileset](https://twitter.com/NNNenov/status/789903180226301953) based on my Castle tileset. Nick uses text output option in the tiled model to reconstruct 3d models in Cinema 4D.
* Sean Leffler implemented the [overlapping model in Rust](https://github.com/sdleffler/collapse).
* rid5x is making an [OCaml version of WFC](https://twitter.com/rid5x/status/782442620459114496).
* I published a very basic [3d tiled model](https://bitbucket.org/mxgmn/basic3dwfc/overview) so people could make their own 3d tilesets without waiting for the full 3d repository.
* I made an [interactive version](https://twitter.com/ExUtumno/status/798571284342837249) of the overlapping model, you can download the GUI executable from the [WFC itch.io page](https://exutumno.itch.io/wavefunctioncollapse).
* [Brian Bucklew](https://github.com/unormal) built a level generation pipeline that applies WFC in multiple passes for the [Caves of Qud](http://store.steampowered.com/app/333640) game: [1](https://twitter.com/unormal/status/805987523596091392), [2](https://twitter.com/unormal/status/808566029387448320), [3](https://twitter.com/unormal/status/808523056259993601), [4](https://twitter.com/unormal/status/808523493994364928), [5](https://twitter.com/unormal/status/808519575264497666), [6](https://twitter.com/unormal/status/808519216185876480), [7](https://twitter.com/unormal/status/808795396508123136), [8](https://twitter.com/unormal/status/808860105093632001), [9](https://twitter.com/unormal/status/809637856432033792), [10](https://twitter.com/unormal/status/810239794433425408), [11](https://twitter.com/unormal/status/811034574973243393), [12](https://twitter.com/unormal/status/811720423419314176), [13](https://twitter.com/unormal/status/811034037259276290), [14](https://twitter.com/unormal/status/810971337309224960), [15](https://twitter.com/unormal/status/811405368777723909), [16](https://twitter.com/ptychomancer/status/812053801544757248), [17](https://twitter.com/unormal/status/812159308263788544), [18](https://twitter.com/unormal/status/812158749838340096), [19](https://twitter.com/unormal/status/814569437181476864), [20](https://twitter.com/unormal/status/814570383189876738), [21](https://twitter.com/unormal/status/819725864623603712).
* [Danny Wynne](https://github.com/dannywynne) implemented a [3d tiled model](https://twitter.com/dwtw/status/810166761270243328).
* Arvi Teikari programmed a [texture synthesis algorithm with the entropy heuristic](http://www.hempuli.com/blogblog/archives/1598) in Lua. Headchant [ported](https://github.com/headchant/iga) it to work with LÖVE.
* Isaac Karth made a [Python port](https://github.com/ikarth/wfc_python) of the overlapping model.
* Oskar Stalberg made an [interactive version](http://oskarstalberg.com/game/wave/wave.html) of the tiled model that runs in the browser.
* [Matt Rix](https://github.com/MattRix) implemented a 3d tiled model ([1](https://twitter.com/MattRix/status/869403586664570880), [2](https://twitter.com/MattRix/status/870999185167962113), [3](https://twitter.com/MattRix/status/871054734018453505), [4](https://twitter.com/MattRix/status/871056805761359872)) and made a 3-dimensional tiled model where one of the dimensions is time ([1](https://twitter.com/MattRix/status/872674537799913472), [2](https://twitter.com/MattRix/status/872648369625325568), [3](https://twitter.com/MattRix/status/872645716660891648), [4](https://twitter.com/MattRix/status/872641331956518914)).
* [Nick Nenov](https://github.com/NNNenov) made a [visual guide](https://www.dropbox.com/s/zeiat1w8zre9ro8/Knots%20breakdown.png?dl=0) to the tile symmetry system.
* [Isaac Karth](https://github.com/ikarth) and [Adam M. Smith](https://github.com/rndmcnlly) wrote a [research paper](https://adamsmith.as/papers/wfc_is_constraint_solving_in_the_wild.pdf) where they formulate WFC as an ASP problem, use general constraint solver [clingo](https://github.com/potassco/clingo) to generate bitmaps, experiment with global constraints, trace WFC's history and give detailed explanation of the algorithm.
* Sylvain Lefebvre made a [C++ implementation](https://github.com/sylefeb/VoxModSynth) of 3d model synthesis, described the thought process of designing a sample and provided an example where adjacency constraints ensure that the output is connected (walkable).
* I generalized 3d WFC to work with cube symmetry group and made a tileset that generates [Escheresque scenes](https://twitter.com/ExUtumno/status/895684431477747715).
* There are many ways to visualize partially observed wave states. In the code, color values of possible options are averaged to produce the resulting color. Oskar Stalberg [shows](https://twitter.com/OskSta/status/863019585162932224) partially observed states as semi-transparent boxes, where the box is bigger for a state with more options. In the voxel setting I [visualize](https://twitter.com/ExUtumno/status/900395635412787202) wave states with per-voxel voting.
* Remy Devaux implemented the tiled model in PICO-8 and wrote an [article](https://trasevol.dog/2017/09/01/di19/) about generation of coherent data with the explanation of WFC.
* For the upcoming game [Bad North](https://www.badnorth.com/) Oskar Stalberg [uses](https://twitter.com/OskSta/status/917405214638006273) a heuristic that tries to select such tiles
that the resulting observed zone is navigable at each step.
* William Manning [implemented](https://github.com/heyx3/easywfc) the overlapping model in C# with the primary goal of making code readable, and provided it with WPF GUI.
* [Joseph Parker](https://gist.github.com/selfsame) wrote a WFC [tutorial](http://www.procjam.com/tutorials/wfc/) for Procjam 2017.
* [Aman Tiwari](https://github.com/aman-tiwari) formulated the connectivity constraint as an [ASP problem](https://gist.github.com/aman-tiwari/8a7b874cb1fd1270adc203b2af293f4c) for clingo.
* MatveyK programmed a [3d overlapping model](https://github.com/MatveyK/Kazimir).
* [Sylvain Lefebvre](https://github.com/sylefeb), [Li-Yi Wei](https://github.com/1iyiwei) and [Connelly Barnes](https://github.com/connellybarnes) are [investigating](https://hal.archives-ouvertes.fr/hal-01706539/) the possibility of hiding information inside textures. They made a [tool](https://members.loria.fr/Sylvain.Lefebvre/infotexsyn/) that can encode text messages as WFC tilings and decode them back. This technique allows to use WFC tilings as QR codes.
* [Mathieu Fehr](https://github.com/math-fehr) and [Nathanael Courant](https://github.com/Ekdohibs) significantly [improved](https://github.com/math-fehr/fast-wfc) the running time of WFC, by an order of magnitude for the overlapping model.

## How to build
WFC is a console application that depends only on the standard library. Build instructions from the community for various platforms can be found in the [relevant issue](https://github.com/mxgmn/WaveFunctionCollapse/issues/3). Casey Marshall made a [pull request](https://github.com/mxgmn/WaveFunctionCollapse/pull/18) that makes using the program with the command line more convenient and includes snap packaging.

## Credits
Some samples are taken from the games Ultima IV and [Dungeon Crawl](https://github.com/crawl/crawl). Circles tileset is taken from [Mario Klingemann](https://twitter.com/quasimondo/status/778196128957403136). Idea of generating integrated circuits was suggested to me by [Moonasaur](https://twitter.com/Moonasaur/status/759890746350731264) and their style was taken from Zachtronics' [Ruckingenur II](http://www.zachtronics.com/ruckingenur-ii/). Cat overlapping sample is taken from the Nyan Cat video, Qud sample was made by [Brian Bucklew](https://github.com/unormal), Magic Office + Spirals samples - by rid5x, Colored City + Link + Link 2 + Mazelike + Red Dot + Smile City overlapping samples - by Arvi Teikari. Summer tileset was made by Hermann Hillmann. Voxel models were rendered in [MagicaVoxel](http://ephtracy.github.io/).

<p align="center"><img alt="second collage" src="http://i.imgur.com/CZsvnc7.png"></p>
<p align="center"><img alt="voxel perspective" src="http://i.imgur.com/RywXCHn.png"></p>
