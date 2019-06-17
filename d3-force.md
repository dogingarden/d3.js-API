# d3-force

该模块实现了一个 [velocity Verlet](https://en.wikipedia.org/wiki/Verlet_integration) 数字积分 for 模拟粒子上的物理力. 该模拟很简单: 它为每一步都假设一个恒定的单位时间步长 Δ*t* = 1 , 和每一个粒子有一个恒定的单位质量 *m* = 1 . 结果, 作用在一个粒子上的力 *F* 等于在恒定时间间隔Δ*t*上的一个恒定加速度 *a* , 可以简单地通过增加粒子的速度来模拟，这个速度会被叠加到粒子的位置上. 

译者注：Verlet算法是经典力学（牛顿力学）中的一种最为普遍的积分方法，被广泛运用在分子运动模拟（Molecular Dynamics Simulation），行星运动以及织物变形模拟等领域。Verlet算法要解决的问题是，给定粒子t时刻的位置r和速度v，得到t+dt时刻的位置r(t+dt)和速度v(t+dt)。最简单的方法是前向计算（考虑当前和未来）的速度位移公式，也就是显式欧拉方法，但精度不够，且不稳定。Verlet积分是一种综合过去、现在和未来的计算方法（居中计算），精度为O(4), 稳定度好，且计算复杂度不比显式欧拉方法高多少。

在信息可视化领域，物理模拟在[网络](http://bl.ocks.org/mbostock/ad70335eeef6d167bc36fd3c04378048)和[层级结果](http://bl.ocks.org/mbostock/95aa92e2f4e8345aaa55a4a94d41ce37)领域非常实用!

[<img alt="Force Dragging III" src="https://raw.githubusercontent.com/d3/d3-force/master/img/graph.png" width="420" height="219">](http://bl.ocks.org/mbostock/ad70335eeef6d167bc36fd3c04378048)[<img alt="Force-Directed Tree" src="https://raw.githubusercontent.com/d3/d3-force/master/img/tree.png" width="420" height="219">](http://bl.ocks.org/mbostock/95aa92e2f4e8345aaa55a4a94d41ce37)

你还可以模拟待用碰撞的圆圈以及圆盘，例如 [bubble charts](http://www.nytimes.com/interactive/2012/09/06/us/politics/convention-word-counts.html) 或者 [beeswarm plots](http://bl.ocks.org/mbostock/6526445e2b44303eebf21da3b6627320):

[<img alt="Collision Detection" src="https://raw.githubusercontent.com/d3/d3-force/master/img/collide.png" width="420" height="219">](http://bl.ocks.org/mbostock/31ce330646fa8bcb7289ff3b97aab3f5)[<img alt="Beeswarm" src="https://raw.githubusercontent.com/d3/d3-force/master/img/beeswarm.png" width="420" height="219">](http://bl.ocks.org/mbostock/6526445e2b44303eebf21da3b6627320)

你可以将它作为基本的物理引擎，比如模拟布料:

[<img alt="Force-Directed Lattice" src="https://raw.githubusercontent.com/d3/d3-force/master/img/lattice.png" width="480" height="250">](http://bl.ocks.org/mbostock/1b64ec067fcfc51e7471d944f51f1611)

使用本模块, 为一个 [nodes](#simulation_nodes)数组建立一个 [simulation](#simulation) , 组成需要的 [forces](#simulation_force). 然后 [监听](#simulation_on) tick 事件，以便在你需要的图形系统（Canvas or SVG）中更新节点并对其进行渲染.

## 安装

If you use NPM, `npm install d3-force`. Otherwise, download the [latest release](https://github.com/d3/d3-force/releases/latest). You can also load directly from [d3js.org](https://d3js.org), either as a [standalone library](https://d3js.org/d3-force.v2.min.js) or as part of [D3](https://github.com/d3/d3). AMD, CommonJS, and vanilla environments are supported. In vanilla, a `d3_force` global is exported:

```html
<script src="https://d3js.org/d3-dispatch.v1.min.js"></script>
<script src="https://d3js.org/d3-quadtree.v1.min.js"></script>
<script src="https://d3js.org/d3-timer.v1.min.js"></script>
<script src="https://d3js.org/d3-force.v2.min.js"></script>
<script>

var simulation = d3.forceSimulation(nodes);

</script>
```

[Try d3-force in your browser.](https://beta.observablehq.com/@mbostock/d3-force-directed-graph)

## API参考

### Simulation模拟

<a name="forceSimulation" href="#forceSimulation">#</a> d3.<b>forceSimulation</b>([<i>nodes</i>]) （<b>力模拟</b>）[<>](https://github.com/d3/d3-force/blob/master/src/simulation.js "Source")

为一组 [*nodes*](#simulation_nodes)创建一个新的力模拟，但是没有[forces](#simulation_force). 如果 *nodes* 没有被指定, 默认值为一个空数组. 这个模拟器自动 [starts](#simulation_restart); 当模拟启动时可以使用 [*simulation*.on](#simulation_on) 来监听tick事件. 如果你想手动启动模拟, 调用 [*simulation*.stop](#simulation_stop), 然后按照需要调用 [*simulation*.tick](#simulation_tick).

<a name="simulation_restart" href="#simulation_restart">#</a> <i>simulation</i>.<b>restart</b>() （重启）[<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L86 "Source")

重启力模拟器内部的计时器并返回这个计时器. 结合 [*simulation*.alphaTarget](#simulation_alphaTarget) 或者 [*simulation*.alpha](#simulation_alpha), 这个方法可以用来在交互期间 “reheat（重加热）” 模拟器, 例如拖拽一个节点, 或者在暂停后 [*simulation*.stop](#simulation_stop)重新启动模拟.

<a name="simulation_stop" href="#simulation_stop">#</a> <i>simulation</i>.<b>stop</b>() （停止）[<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L90 "Source")

停止模拟器内部计时器, 如果正在运行，则同时返回这个模拟器. 如果计时器已经被停止了，则什么都不做. 这个方法对手动运行模拟非常有用; 参考 [*simulation*.tick](#simulation_tick).

<a name="simulation_tick" href="#simulation_tick">#</a> <i>simulation</i>.<b>tick</b>([<i>iterations</i>])（节动） [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L38 "Source")

通过给定的 *迭代*次数手动执行模拟, 并返回模拟器. 如果 *迭代* 没有被指定, 默认为1(单步).

每次迭代过程, 将当前的 [*alpha*](#simulation_alpha) 增加 ([*alphaTarget*](#simulation_alphaTarget) - *alpha*) × [*alphaDecay*](#simulation_alphaDecay); 然后使用每一个注册的 [force](#simulation_force), 传递新的 *alpha*; 然后通过 *velocity* × [*velocityDecay*](#simulation_velocityDecay)递减每一个 [node](#simulation_nodes)’的速度; 最后通过 *velocity*递增/修改每个节点的位置.

此方法并不会调度事件 [events](#simulation_on); 只有在 [creation](#forceSimulation) 时自动启动模拟或者调用 [*simulation*.restart](#simulation_restart)的时候，内部计时器才会调度事件. 模拟启动时候自然的ticks数为 ⌈*log*([*alphaMin*](#simulation_alphaMin)) / *log*(1 - [*alphaDecay*](#simulation_alphaDecay))⌉; 默认状态下为 300.

此方法可以结合 [*simulation*.stop](#simulation_stop) 来计算静态力布局 [static force layout](https://bl.ocks.org/mbostock/1667139). 对于大型图, 应当在[in a web worker](https://bl.ocks.org/mbostock/01ab2e85e8727d6529d20391c0fd9a16) 计算static layouts静态布局以避免冻结用户界面.

<a name="simulation_nodes" href="#simulation_nodes">#</a> <i>simulation</i>.<b>nodes</b>([<i>nodes</i>])（节点） [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L94 "Source")

如果指定了*节点nodes*, 则将模拟节点设置为指定的对象数组, 必要时初始化它们的初始位置和速度, 然后初始化[re-initializes](#force_initialize) 任何约束力[forces](#simulation_force); 返回模拟器. 如果*节点nodes* 未被指定, 则返回指定构造器 [constructor](#forceSimulation)的模拟节点数组.

每个节点*node* 必须是一个对象. 模拟器分配以下属性:

* `index` - 从零开始的*nodes*索引
* `x` - 节点的当前*x*-位置
* `y` - 节点的当前*y*-位置
* `vx` - 节点的当前*x*-速度
* `vy` - 节点的当前*y*-速度

位置 ⟨*x*,*y*⟩ 和速度 ⟨*vx*,*vy*⟩ 随后可以通过力 [forces](#forces) 和模拟器修改. 如果 *vx* 或者 *vy* 是 NaN, 速度初始化为 ⟨0,0⟩. 如果 *x* 或者 *y* 是 NaN, 则位置以 叶序排序[phyllotaxis arrangement](http://bl.ocks.org/mbostock/11478058)初始化, 因此选择一个确定值, 确保在原点周围的确定的均匀分布.

要在给定位置固定节点，您可以指定另外两个属性：

* `fx` - 节点的固定的 *x*-位置
* `fy` - 节点的固定的 *y*-位置

在每个节点节动 [tick](#simulation_tick)结束时, 在任何力应用后, 具有已定义 *node*.fx 的节点会将 *node*.x 重置为此值并将 *node*.vx 设置为0; 同样地, 具有已定义 *node*.fy 的节点会将 *node*.y 重置为该值并将*node*.vy 设置为0. 要解除先前固定的节点，请将node.fx和node.fy设置为null，或删除这些属性.

如果修改了指定的节点数组，例如在模拟中添加或删除节点时，必须使用新的（或更改的）数组再次调用此方法，以通知更改模拟和绑定力;模拟不会生成指定数组的defensive copy(防御副本)。

<a name="simulation_alpha" href="#simulation_alpha">#</a> <i>simulation</i>.<b>alpha</b>([<i>alpha</i>])（） [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L98 "Source")

alpha为冷却因子，是一种混合参数，用于组合各种力作用于节点。在每一个tick节拍处递减，并减少每一个力对节点位置的影响，在到达alpha的某个阈值后，力布局停止计算，将图形冻结为希望的最佳布局。
如果给定 *alpha* 的值, 将当前的alpha值设置为 [0,1]区间的一个数并返回当前的模拟器. 如果 *alpha* 值未被指定, 则返回当前的alpha值, 默认为 1.

<a name="simulation_alphaMin" href="#simulation_alphaMin">#</a> <i>simulation</i>.<b>alphaMin</b>([<i>min</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L102 "Source")

如果指定了*min* , 则将*alpha* 的最小值设置为[0,1]范围内的指定数字并返回此模拟. 如果*min* 未被指定, 则返回 *alpha* 的最小值, 默认为 0.001.当当前[*alpha*](#simulation_alpha)小于 *alpha*最小值时，模拟的内部计时器停止. 默认的 [alpha decay rate](#simulation_alphaDecay)衰减速率~0.0228 对应300 次迭代.

<a name="simulation_alphaDecay" href="#simulation_alphaDecay">#</a> <i>simulation</i>.<b>alphaDecay</b>([<i>decay</i>])（alpha衰减速率） [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L106 "Source")

如果指定了衰减*decay* , 则将 [*alpha*](#simulation_alpha) 衰减速率设置为范围[0,1]中的指定数字，并返回模拟. 如果*decay*衰减速率未指定, 返回当前的 *alpha* 衰减速率,默认为 0.0228… = 1 - *pow*(0.001, 1 / 300) ，这里的0.001 是默认的[minimum *alpha*](#simulation_alphaMin).

alpha衰减速率决定了当前alpha值向目标值[target *alpha*](#simulation_alphaTarget)插值的速率;因为默认的*alpha*目标值为0,因此默认情况下它控制了模拟的冷却速度. 衰减速率越高，模拟到达稳定状态就越快，但有一定风险会停留在局部的最小值;值越低模拟运行时长越长，但通常会收敛到更好的布局上. 让布局永远以当前的 *alpha*值运行, 将 *decay* 衰减速率设置为0; 或者, 将目标 [target *alpha*](#simulation_alphaTarget) 设定大于 [minimum *alpha*](#simulation_alphaMin).

<a name="simulation_alphaTarget" href="#simulation_alphaTarget">#</a> <i>simulation</i>.<b>alphaTarget</b>([<i>target</i>])（alpha目标） [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L110 "Source")

如果指定了目标，则将当前目标alpha设置为范围[0,1]中的指定数字，并返回此模拟。如果未指定目标，则返回当前目标alpha值，默认值为0

<a name="simulation_velocityDecay" href="#simulation_velocityDecay">#</a> <i>simulation</i>.<b>velocityDecay</b>([<i>decay</i>]) （衰减速率）[<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L114 "Source")

如果指定了*decay*, 设定速度衰减因子velocity decay factor 为 [0,1]区间内的指定数字并返回此模拟. 如果 *decay* 未被指定, 返回当前的速度衰减因子velocity decay factor,默认为0.4. 衰减因子decay factor 类似于大气摩擦; 在节拍 [tick](#simulation_tick)期间应用任何力之后, 每一个节点的速度乘以 1 - *decay*.类似 [alpha decay rate](#simulation_alphaDecay), 小的速度衰减速率可能会导致更好的收敛结果, 但是存在数值不稳定和震荡风险.

<a name="simulation_force" href="#simulation_force">#</a> <i>simulation</i>.<b>force</b>(<i>name</i>[, <i>force</i>]) （力）[<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L118 "Source")

如果指定了力*force*, 则为指定的名称*name*指定力[force](#forces) 并返回此模拟. 如果*force* 未指定, 则返回具有指定名称的力, 如果没有这个力的话就返回undefined. (默认情况下, 新的模拟没有力.) 例如, 对于一个图的布局创建一个新的模拟,你可以这样做:

```js
var simulation = d3.forceSimulation(nodes)
    .force("charge", d3.forceManyBody())
    .force("link", d3.forceLink(links))
    .force("center", d3.forceCenter());
```

通过指定的名称 *name*移除力, 可以通过传递null值给 *force*. 例如, 移除charge（电荷） 力:

```js
simulation.force("charge", null);
```

<a name="simulation_find" href="#simulation_find">#</a> <i>simulation</i>.<b>find</b>(<i>x</i>, <i>y</i>[, <i>radius</i>]) （搜索）[<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L122 "Source")

返回与⟨*x*,*y*⟩半径*radius*内最近的节点 . 如果 *radius* 未被指定, 默认为无穷远. 如果搜索区域中没有节点,则返回undefined.

<a name="simulation_on" href="#simulation_on">#</a> <i>simulation</i>.<b>on</b>(<i>typenames</i>, [<i>listener</i>])(监听) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L145 "Source")

如果指定了监听器*listener* , 为指定的类型名称*typenames*设置事件监听器 *listener* 并返回当前模拟. 如果已经为同一名称注册了监听事件，则在添加新的监听之前，删除已有的监听器.如果监听器*listener* 为空（新分配的）,移除给定名称 *typenames*的当前监听器（如果有的话）.如果没有指定监听器*listener* ,则返回第一个与指定名称*typenames*相匹配的监听器（如果有的话）. 当一个指定事件被调用, 每一个 *listener* 都会被调用，其`this`值为当前的模拟，即以当前模拟作为调用函数的上下文.

 *typenames*类型名是一个字符串包含一个或多个由空格分隔*typename*类型名. 每一个 *typename* 都是一个*type*, 可选地跟一个句点 (`.`) 以及一个 *name*,例如 `tick.foo` 和`tick.bar`; 这个名字允许多个监听器注册同一个 *type*类型. *type*类型必须是以下之一:

* `tick` - 在模拟内部计时器的每个节拍之后.
* `end` - 在模拟计时器在 *alpha* < [*alphaMin*](#simulation_alphaMin)停止后.

注意手动调用[*simulation*.tick](#simulation_tick) 时不会触发 *tick* 事件; 事件仅仅由内部计时器调度，用于模拟的交互式渲染. 要影响模拟的过程, 注册 [forces](#simulation_force) 而不是修改节拍事件监听器内部的节点位置或速度.

更多细节请参考 [*dispatch*.on](https://github.com/d3/d3-dispatch#dispatch_on).

### Forces（力）

 *force* 力只是一种修改节点位置或速度的函数; 在这种情况下, 一个 *force*可以施加经典的物理力，例如电荷力或重力,或者它可以解决几何约束问题, 例如将节点保持在边界框内或使得链接的节点保持固定的距离. 例如, 将节点移向远点<0,0>的简单定位力可以实现为:

```js
function force(alpha) {
  for (var i = 0, n = nodes.length, node, k = alpha * 0.1; i < n; ++i) {
    node = nodes[i];
    node.vx -= node.x * k;
    node.vy -= node.y * k;
  }
}
```

力通常读取节点当前的位置⟨*x*,*y*⟩ 然后加上(或减去)节点的速度 ⟨*vx*,*vy*⟩. 然而, 力也可预测节点的下一个位置⟨*x* + *vx*,*y* + *vy*⟩；这对于通过 [iterative relaxation（迭代松弛）](https://en.wikipedia.org/wiki/Relaxation_\(iterative_method\))来解决几何约束是必要的. 力也可以直接修改位置，这有时可以避免向模拟添加能量，例如在视窗中重新定位模拟时。

模拟通常要根据需求组合多个力。这个模块提供几个供您选择:

* [Centering（向心力）](#centering)
* [Collision（碰撞）](#collision)
* [Links（链接）](#links)
* [Many-Body（多体）](#many-body)
* [Positioning（定位）](#positioning)

Forces可以选择实现 [*force*.initialize](#force_initialize) 来接收模拟的节点数组.

<a name="_force" href="#_force">#</a> <i>force</i>(<i>alpha</i>) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L47 "Source")

应用此力, 可以选择观察指定的 *alpha*. 通常, 将力应用于先前传递给[*force*.initialize](#force_initialize)的节点数组, 然而,一些力也许可以应用于节点子集, 或行为不同. 例如, [d3.forceLink](#links) 适用于每个链接的源和目标.

<a name="force_initialize" href="#force_initialize">#</a> <i>force</i>.<b>initialize</b>(<i>nodes</i>) [<>](https://github.com/d3/d3-force/blob/master/src/simulation.js#L77 "Source")

将节点数组*nodes*分配给此力. 当通过[*simulation*.force](#simulation_force)将力绑定到模拟以及当此模拟节点通过 [*simulation*.nodes](#simulation_nodes)更改时，将调用此方法. 力可以在初始化期间执行必要的工作,例如评估每个节点的参数，以避免在每次施加力时重复执行工作.

#### Centering（向心力）

向心力均匀地移动节点，使得所有节点的平均位置（如果所有节点具有相同的重量，则为质心）位于给定位置⟨[*x*](#center_x),[*y*](#center_y)⟩. 该力修改每个应用节点的位置;它不会改变速度,因为这样做通常会导致节点过冲并围绕预定中心震荡. 此力有助于保持节点保持在视窗的中心。和定位力[positioning force](#positioning)不同,它不会扭曲它们的相对位置.

<a name="forceCenter" href="#forceCenter">#</a> d3.<b>forceCenter</b>([<i>x</i>, <i>y</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/center.js#L1 "Source")

使用特定的[*x*-](#center_x) 和[*y*-](#center_y) 坐标创建一个新的向心力. 如果 *x* 和 *y* 没有被指定, 默认为⟨0,0⟩.

<a name="center_x" href="#center_x">#</a> <i>center</i>.<b>x</b>([<i>x</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/center.js#L27 "Source")

如果 *x* 被指定, 设定中心位置的 *x*-coordinate 为特定的数值并返回这个力. 如果 *x* 没有被指定, 返回当前的 *x*-coordinate, 默认为0.

<a name="center_y" href="#center_y">#</a> <i>center</i>.<b>y</b>([<i>y</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/center.js#L31 "Source")

如果 *y* 被指定, 设定中心位置的 *y*-coordinate 为特定的数值并返回这个力. 如果 *y* 没有被指定, 返回当前的 *y*-coordinate, 默认为0.

#### Collision（碰撞）

The collision force（碰撞力） 将节点视为有半径 [radius](#collide_radius)的圆, 而不是点,并阻止节点发生重叠. 更正式地说, 两个节点 *a* 和 *b* 是分离的，所以 *a* 和 *b* 之间的距离至少为 *radius*(*a*) + *radius*(*b*). 为了减少抖动, 默认有一个 “soft” constraint（“软”约束），该约束具有可配置的[strength](#collide_strength) 和 [iteration count](#collide_iterations).

<a name="forceCollide" href="#forceCollide">#</a> d3.<b>forceCollide</b>([<i>radius</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/collide.js "Source")

穿件具有指定半径 [*radius*](#collide_radius)的新的圆碰撞力. 如果 *radius* 未被指定, 默认所有节点都是常量1.

<a name="collide_radius" href="#collide_radius">#</a> <i>collide</i>.<b>radius</b>([<i>radius</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/collide.js#L86 "Source")

如果 *radius* 被指定, 将半径访问器设定为指定的数字或函数, 重新计算每个节点的半径访问器, 并返回此力. 如果 *radius* 没有被指定, 返回当前的半径访问器, 默认为:

```js
function radius() {
  return 1;
}
```

为模拟中的每个节点 [node](#simulation_nodes) 调用半径访问器, 传递节点 *node* 以及其从零开始的索引 *index*. 然后将结果数存储在内部, 这样每个节点的半径仅在初始化力时或者使用新半径调用此方法时重新计算，而不是在每次应用力时调用.

<a name="collide_strength" href="#collide_strength">#</a> <i>collide</i>.<b>strength</b>([<i>strength</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/collide.js#L82 "Source")

如果指定*strength* , 将力强度设置为范围[0,1]中的指定数字，并返回此力.如果未指定*strength* , 则返回默认为 0.7的当前强度.

节点的重叠可以通过迭代松弛来解决. 对于每一个节点, 确定在下一个节拍被预测会发生重叠的其它节点 (使用预期位置 ⟨*x* + *vx*,*y* + *vy*⟩); 然后修改节点的速度，将节点从重叠区域推出. 速度的变化被力的强度所抑制，这样同时重叠的解决就可以混合在一起找到一个稳定的解.

<a name="collide_iterations" href="#collide_iterations">#</a> <i>collide</i>.<b>iterations</b>([<i>iterations</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/collide.js#L78 "Source")

如果指定了迭代*iterations*, 则将每个应用程序的迭代次数设置为指定的次数，并返回此力。 如果未指定 *iterations* , 则返回默认为1的当前迭代计数。增加迭代次数大大增加了约束的刚性，避免了节点的部分重叠，同时也增加了计算力的运行时成本。
#### Links

链接力根据所需的链接距离将链接节点推到一起或分开 [link distance](#link_distance). 力的强度与链接节点的距离和目标距离之间的差成正比，类似于弹簧力.

<a name="forceLink" href="#forceLink">#</a> d3.<b>forceLink</b>([<i>links</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/link.js "Source")

使用指定的链接和默认参数创建新的链接力。如果未指定*links*链接，则默认为空数组。

<a name="link_links" href="#link_links">#</a> <i>link</i>.<b>links</b>([<i>links</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/link.js#L92 "Source")

如果指定 *links*,设置与此力关联的链接数组, 重新计算每个链接的距离 [distance](#link_distance) 与强度 [strength](#link_strength) 参数, 并返回此力. 如果未指定链接*links* , 则返回链接的当前数组, 默认为空数组.

每一个链接都是具有以下属性的对象:

* `source` - 链接的原始节点; 参见 [*simulation*.nodes](#simulation_nodes)
* `target` - 链接的目标节点; 参见 [*simulation*.nodes](#simulation_nodes)
* `index` - 从零开始的 *links*的节点, 按此方法分配。


方便起见, 可以使用数字或字符串标识符而不是对象引用来初始化链接的源和目标属性; 参考 [*link*.id](#link_id). 当链接力被初始化 [initialized](#force_initialize) (或重新初始化, 如节点或链接更改时),任何非对象的 *link*.source 或 *link*.target 属性都会被替换为具有给定标识符的响应节点的对象引用。

如果修改了指定的链接*links*数组, 例如在模拟中添加或删除链接时，必须使用新的（或更改的）数组再次调用此方法，以通知更改力；该力不会对指定的数组进行防御性复制。

<a name="link_id" href="#link_id">#</a> <i>link</i>.<b>id</b>([<i>id</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/link.js#L96 "Source")

如果指定了id,则将节点id访问器设置为指定的函数并返回此力。如果没有指定id，返回当前的节点id访问器，默认node.index 数字编号:

```js
function id(d) {
  return d.index;
}
```

默认的id访问器允许将每一个链接的源和目标指定为节点 [nodes](#simulation_nodes) 数组中从0开始的索引. 例如:

```js
var nodes = [
  {"id": "Alice"},
  {"id": "Bob"},
  {"id": "Carol"}
];

var links = [
  {"source": 0, "target": 1}, // Alice → Bob
  {"source": 1, "target": 2} // Bob → Carol
];
```

现在考虑另一个返回字符串的id访问器:

```js
function id(d) {
  return d.id;
}
```

使用这个访问器，你可以使用源与目标的名称：

```js
var nodes = [
  {"id": "Alice"},
  {"id": "Bob"},
  {"id": "Carol"}
];

var links = [
  {"source": "Alice", "target": "Bob"},
  {"source": "Bob", "target": "Carol"}
];
```

当使用JSON表示图的时候，这特别有用, 因为JSON不允许索引. 参见 [this example](http://bl.ocks.org/mbostock/f584aa36df54c451c94a9d0798caed35).

每当力初始化时，都会为每个节点调用id访问器, 例如当节点[nodes](#simulation_nodes)或链接 [links](#link_links) 更改, 会传递给节点及其从0开始的索引.

<a name="link_distance" href="#link_distance">#</a> <i>link</i>.<b>distance</b>([<i>distance</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/link.js#L108 "Source")

如果指定了距离*distance* , 则将距离访问器设置为指定的数字或函数, 重新计算每个链接的距离访问器,并返回此力. 如果距离 *distance* 未被指定, 则返回当前的距离访问器, 默认为:

```js
function distance() {
  return 30;
}
```

对每个链接 [link](#link_links)调用距离访问器, 并传递链接 *link* 及其从0开始的索引 *index*. 然后将生成的数字存储在内部, 这样只有在力初始化或者使用新的距离调用此方法时，而不是在每次应用力时，才会重新计算每个链接的距离。

<a name="link_strength" href="#link_strength">#</a> <i>link</i>.<b>strength</b>([<i>strength</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/link.js#L104 "Source")

如果指定了强度*strength*，则将强度访问器设置为指定的数字或函数，重新计算每个链接的强度访问器，并返回此力。如果未指定strength，则返回当前*strength*访问器，默认值为：

```js
function strength(link) {
  return 1 / Math.min(count(link.source), count(link.target));
}
```

其中 *count*(*node*) 是一个函数，返回给定节点作为源或目标的链接数. 之所以选择此默认值，是因为它会自动降低连接到heavily-connected（重链接）节点的链接的强度，从而提高稳定性.

对每个链接 [link](#link_links)调用强度访问器, 并传递链接 *link*及其从0开始的索引 *index*. 然后将生成的数字存储在内部，这样，只有在初始化力或使用新的强度调用此方法时，而不是在每次应用力时，才会重新计算每个链接的强度*strength*.

<a name="link_iterations" href="#link_iterations">#</a> <i>link</i>.<b>iterations</b>([<i>iterations</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/link.js#L100 "Source")

如果指定了迭代*iterations* , 则将每个应用的迭代次数设定为指定的次数. 如果 *iterations* 未被指定, 返回默认为1的当前迭代次数.增加迭代次数会大大增加约束的刚度，对于 [复杂结构，如格子](http://bl.ocks.org/mbostock/1b64ec067fcfc51e7471d944f51f1611)会很有用, 当也会增加计算力的运行时间成本.

#### Many-Body

多体 many-body (或*n*体 *n*-body) 力在所有点[nodes](#simulation_nodes)之间相互作用. 如果[strength](#manyBody_strength) 为正，它可以用来模拟重力(引力) ；如果强度（strength）为负，则可以模拟静电荷力 (排斥) . 该实现使用四叉树和 [Barnes–Hut approximation](https://en.wikipedia.org/wiki/Barnes–Hut_simulation) 来大幅度提升性能; 精确度可以使用 [theta](#manyBody_theta) 参数来进行控制.

与链接只影响两个被连接的节点不同，电荷力（charge force）是全局的：每个节点都会影响其它节点，即使它们位于断开的子图上.

<a name="forceManyBody" href="#forceManyBody">#</a> d3.<b>forceManyBody</b>() [<>](https://github.com/d3/d3-force/blob/master/src/manyBody.js "Source")

使用默认参数创建新的many-body力.

<a name="manyBody_strength" href="#manyBody_strength">#</a> <i>manyBody</i>.<b>strength</b>([<i>strength</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/manyBody.js#L97 "Source")

如果指定了强度*strength* , 将*strength*访问器设定为特定的数字或函数, 为每个节点计算强度访问器, 并返回此力. 正值使得节点互相吸引, 类似重力； 负值使节点互相排斥，类似静电荷力. 如果未指定*strength* , 返回当前的strength访问器, 默认为:

```js
function strength() {
  return -30;
}
```

对模拟中的每个节点[node](#simulation_nodes)调用强度访问器, 并传递节点 *node* 及其从0开始的索引 *index*. 然后将生成的数字存储在内部, 这样只有在力初始化或这个方法被调用时，而不是每次应用此力时，才会重新计算每个点解的强度*strength*。

<a name="manyBody_theta" href="#manyBody_theta">#</a> <i>manyBody</i>.<b>theta</b>([<i>theta</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/manyBody.js#L109 "Source")

如果指定*theta* , 将Barnes–Hut 估计标准设置为指定的数字并返回该力。 如果未指定 *theta*, 返回当前值,默认为 0.9.

为了加速计算, 这个力实现了 [Barnes–Hut 估计](http://en.wikipedia.org/wiki/Barnes–Hut_simulation) ，这里每次应用取 O(*n* log *n*) ， *n* 是[nodes](#simulation_nodes)的数量. 对每次应用,  [四叉树quadtree](https://github.com/d3/d3-quadtree) 存储当前节点的位置; 然后对于每个节点, 计算所有其它节点对该节点的合力. 对于较远的一组节点, 可以将其视为更大的一个节点来近似计算电荷力。  *theta* 参数决定了近似值的准确性: 如果 四叉树单元的宽度*w*与节点到单元质心中心距离*l*的比值 *w* / *l* 小于*theta*, 所有给定单元中节点被看作一个节点而不是每个都是一个个体。

<a name="manyBody_distanceMin" href="#manyBody_distanceMin">#</a> <i>manyBody</i>.<b>distanceMin</b>([<i>distance</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/manyBody.js#L101 "Source")

如果指定了*distance*，设置节点间最小距离的时候应当考虑力的影响. 如果未指定 *distance* , 返回当前的最小距离, 该值默认为1. 最小距离建立了两个相邻节点之间的力的强度上限，避免了不稳定. 特别是, 如果这两个节点完全重合时，可以避免产生无穷大的力；在这种情况下，力的方向是随机的.

<a name="manyBody_distanceMax" href="#manyBody_distanceMax">#</a> <i>manyBody</i>.<b>distanceMax</b>([<i>distance</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/manyBody.js#L105 "Source")

如果指定了 *distance* , 设置节点间最大距离的时候应当考虑力的影响. 如果未指定 *distance* , 返回当前的最大距离, 默认为无穷大. 指定的有限大的值可以提升性能并使得生成结果更加局限在小范围内.

#### Positioning

[*x*](#forceX)- 和 [*y*](#forceY)-定位力将节点沿着给定维度推向所需位置，该力具有可配置的强度. [*radial*](#forceRadial) force （径向力）是相似的, 只是它将节点推向给定圆上最近的点. 力的强度与该节点位置与目标位置的一位长度成正比. 虽然这些力可以用于定位单个节点, 但它们主要用于应用于所有（或大多数节点）的全局力.

<a name="forceX" href="#forceX">#</a> d3.<b>forceX</b>([<i>x</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/x.js "Source")

沿着 *x*-axis创建新的指向给定位置[*x*](#x_x)的定位力. 如果未指定*x* , 默认为 0.

<a name="x_strength" href="#x_strength">#</a> <i>x</i>.<b>strength</b>([<i>strength</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/x.js#L32 "Source")

如果指定了*strength*, 则将强度访问器设定为指定的数字或函数, 重新计算每个节点的强度访问器, 并返回此力.强度 *strength* 决定了节点 *x*-velocity的增速为多少: ([*x*](#x_x) - *node*.x) × *strength*. 例如, 值 0.1 表示节点应该在每次对节点应用力时，从其当前x位置移动十分之一到目标x位置. 值越大，节点移动到目标位置的速度越快, 通常会牺牲其他力或约束. 不建议使用超出范围[0,1]的值.

如果未指定*strength*, 返回当前的强度访问器, 默认为:

```js
function strength() {
  return 0.1;
}
```

对模拟中的每个节点 [node](#simulation_nodes) 调用强度访问器, 并传递该节点 *node* 及其从0开始的索引 *index*. 然后将生成的数据存储在内部, 这样，只有在力初始化或使用新的强度 *strength*参数调用此方法时，而不是每次应用力时，才会重新计算每个节点的强度.

<a name="x_x" href="#x_x">#</a> <i>x</i>.<b>x</b>([<i>x</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/x.js#L36 "Source")

如果指定了 *x* , 则将 *x*-coordinate 访问器设置为指定的数字或函数, 重新计算每个节点的*x*-坐标访问器, 并返回此力. 如果未指定 *x* , 则返回当前的 *x*-访问器, 默认为:

```js
function x() {
  return 0;
}
```

对模拟器中每个节点[node](#simulation_nodes) 调用*x*-访问器, 并传递该 *node* 及其从0开始的索引 *index*. 然后，生成的数字存储在内部，这样每个节点的目标X坐标只能在力初始化或使用新的X调用此方法时重新计算，而不是在每次应用力时重新计算。

<a name="forceY" href="#forceY">#</a> d3.<b>forceY</b>([<i>y</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/y.js "Source")

Creates a new positioning force along the *y*-axis towards the given position [*y*](#y_y). If *y* is not specified, it defaults to 0.

<a name="y_strength" href="#y_strength">#</a> <i>y</i>.<b>strength</b>([<i>strength</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/y.js#L32 "Source")

If *strength* is specified, sets the strength accessor to the specified number or function, re-evaluates the strength accessor for each node, and returns this force. The *strength* determines how much to increment the node’s *y*-velocity: ([*y*](#y_y) - *node*.y) × *strength*. For example, a value of 0.1 indicates that the node should move a tenth of the way from its current *y*-position to the target *y*-position with each application. Higher values moves nodes more quickly to the target position, often at the expense of other forces or constraints. A value outside the range [0,1] is not recommended.

If *strength* is not specified, returns the current strength accessor, which defaults to:

```js
function strength() {
  return 0.1;
}
```

The strength accessor is invoked for each [node](#simulation_nodes) in the simulation, being passed the *node* and its zero-based *index*. The resulting number is then stored internally, such that the strength of each node is only recomputed when the force is initialized or when this method is called with a new *strength*, and not on every application of the force.

<a name="y_y" href="#y_y">#</a> <i>y</i>.<b>y</b>([<i>y</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/y.js#L36 "Source")

If *y* is specified, sets the *y*-coordinate accessor to the specified number or function, re-evaluates the *y*-accessor for each node, and returns this force. If *y* is not specified, returns the current *y*-accessor, which defaults to:

```js
function y() {
  return 0;
}
```

The *y*-accessor is invoked for each [node](#simulation_nodes) in the simulation, being passed the *node* and its zero-based *index*. The resulting number is then stored internally, such that the target *y*-coordinate of each node is only recomputed when the force is initialized or when this method is called with a new *y*, and not on every application of the force.

<a name="forceRadial" href="#forceRadial">#</a> d3.<b>forceRadial</b>(<i>radius</i>[, <i>x</i>][, <i>y</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/radial.js "Source")

[<img alt="Radial Force" src="https://raw.githubusercontent.com/d3/d3-force/master/img/radial.png" width="420" height="219">](https://bl.ocks.org/mbostock/cd98bf52e9067e26945edd95e8cf6ef9)

对以 ⟨[*x*](#radial_x),[*y*](#radial_y)⟩为中心的指定半径 [*radius*](#radial_radius)的圆创建新的定位力 . 如果未指定 *x* 和 *y* , 默认为 ⟨0,0⟩.

<a name="radial_strength" href="#radial_strength">#</a> <i>radial</i>.<b>strength</b>([<i>strength</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/radial.js "Source")

如果指定了*strength*, 则将强度访问器设定为指定的数字或函数, 重新计算每个节点的强度访问器, 并返回此力.强度 *strength* 决定了节点 *x*-velocity和*y*-velocity的增速为多少。例如, 值 0.1 表示节点应该在每次对节点应用力时，从其当前x位置移动十分之一到目标x位置. 值越大，节点移动到目标位置的速度越快, 通常会牺牲其他力或约束. 不建议使用超出范围[0,1]的值.

如果未指定strength，则返回当前strength访问器，默认值为：

```js
function strength() {
  return 0.1;
}
```

为模拟中每个节点调用强度访问器 [node](#simulation_nodes) , 传递节点 *node* 和其从0开始的索引 *index*. 然后在内部存储得到的数字，使得每个节点的强度仅在初始化力时或者以新的力量调用该方法时重新计算，而不是在力的每个应用上调用。

<a name="radial_radius" href="#radial_radius">#</a> <i>radial</i>.<b>radius</b>([<i>radius</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/radial.js "Source")

如果指定了半径 *radius* , 将圆的半径 *radius* 设置为指定的数字或函数, 重新计算每个节点的半径*radius* 访问器，并返回此力. 如果未指定 *radius* , 返回当前的 *radius* 访问器.

在模拟中的每个节点[node](#simulation_nodes)调用 *radius* 访问器, 传递节点 *node* 及其从0开始的索引 *index*. 然后在内部存储得到的数字, 然后在内部存储得到的数字，使得每个节点的目标半径仅在初始化力时或者在使用新半径 *radius* 调用此方法时重新计算，而不是在力的每个应用上调用。

<a name="radial_x" href="#radial_x">#</a> <i>radial</i>.<b>x</b>([<i>x</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/radial.js "Source")

如果指定了x，则将圆心的x坐标设置为指定的数字并返回此力。 如果未指定x，则返回中心的当前x坐标，默认为零。

<a name="radial_y" href="#radial_y">#</a> <i>radial</i>.<b>y</b>([<i>y</i>]) [<>](https://github.com/d3/d3-force/blob/master/src/radial.js "Source")

如果指定了y，则将圆心的y坐标设置为指定的数字并返回此力。 如果未指定y，则返回中心的当前y坐标，默认为零。