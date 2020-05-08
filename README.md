## 2020年队式比赛

加油！~学姐最棒了x~

用数电状态机的思想，大概看作三个状态：【1.无菜肴正在烹饪】【2.有菜肴正在烹饪】【3.手中有成品菜等待提交】

任意一个状态都以进入下移状态为目标，如果捡到了别人做好的菜直接进入第三个状态。

# UPDATE-2020/5/7

## Player 1

起手还是先找最近的食物生产点的食物带到最近的灶台

**【1.无菜肴正在烹饪】**

判断灶台是否可用

（有无黑暗料理或被别人用了，同之前，被别人用了就找下一个灶台更新label，没人就标记为自己的）

更新储存列表

获取任务列表，检索储存列表（优先任务中有的菜，其次任意菜，再次中间产物，如果中间产物已经有了就不再做了），如果返回不是空表，就根据返回的表和坐标制作，进入【状态2】。

如果没有进入状态2，就再找食材

以label标记的灶台位置为基准，计算每个菜肴的性价比（拿空表算缺什么食材，分数/获取时间，越大越好），优先制作性价比最高且没有储存的菜肴。将这个菜肴传入函数，检索缺什么食材，跑过去拿。



**【2.有菜肴正在烹饪】**





**【3.手中有成品菜等待提交】**





## Player 2

主体逻辑不变

# Old-version

### 主player

**【1.无菜肴正在烹饪】**

Q1：手上是否有食材？

无：搜索最近的食材生成点，跑过去拿食材，拿到继续 没拿到-守株待兔-Q1

有：搜索最近的灶台，跑过去，Q2

Q2：检查灶台是否有黑暗料理
    是：尝试拾取
        成功就扔出去，并且标记灶台位置为【据点】方便后面回来，进入Q3。
        失败就说明该灶台正在烹饪，标记该灶台为【别人再用】，然后换一个
        （如果所有的灶台都搜索了一遍，仍然没有空闲灶台，就回到第一个再来。）。
    否：首先标记这个灶台的位置为【据点】，方便以后回来。
        Q3：检查灶台和桌子上是否有食材：
        无：放下手中食材，标记灶台位置，告知队友坐标。
            如果自己手中食材和队友手中食材匹配，直接做，不匹配或者没有：
            如果手里是【麦子1，水稻2，牛奶5】直接做成【面粉20，米饭23】。面粉优先级最高
            如果手里是【面粉20】，直接做成【面条21】
            如果手里是【番茄3】，看看队友有无【鸡蛋4】，有做【番茄炒蛋26】，无做【番茄酱24】
            如果手里是【牛奶5】，直接做成【奶油25】
            如果手里是【山楂9】，直接做成【糖葫芦39】
            其他：先放桌子上……
        有：首先检查是否有【成品菜肴26-51】，如果有，拿起来进入**【3.手中有成品菜等待提交】**
            接着检查桌子+灶台上的食材与手上食材是否匹配（如果能做出任意成品，就直接一起放下去，否则先扔到桌子上）
            （如果一共有n个食材，优先匹配n的菜谱，再分别单独看n-1个的，n-2……
            其次检测所有的食材中是否有【中间食材20-25】，如果有，下一步中不重复制作相同的中间食材。
            最后检测手里的食材是不是能直接做出产品(就是上面那几行)，可以就把别人的扔到桌子上，把自己的放进去。
            如果以上两个都不行，那就把别人的和自己的都扔到桌子上。  

note:所有“检测是否匹配”都包含队友手里的食材，打算设计成主从结构(数电洗脑x)？就是主逻辑在一个人身上，队友只是听从安排？

Q3：刚刚开始烹饪了吗？
否：回到Q1抢食材，只不过拿到食材后先回到刚刚标记的灶台
    除非它已经开始烹饪，则取消标记，换个据点（不断跑直到碰到没有正在烹饪的灶台）
是：进入**【2.有菜肴正在烹饪】**
    计时并计算｛完成时间，保护时间（*1.25）,overcooked时间（*1.5）｝，标记刚刚烹饪了啥
    如果刚刚做的是【中间产物20-25】，计算可能的最终产物（如【20面粉】→【21面条】→【11牛肉+21面条】=【28青青牛拉】等等）
        只额外需要一个食材的（比如刚刚的面），一旦拿到这个食材就**一直拿在手上**，回到据点附近，不要再跑了。
        额外需要两个食材的（如盖浇饭），无所谓，正常跑。
    两人继续搜索食材（找附近最近的生成点），回来放到桌子上（其他人抢就抢吧）
    考虑到大多数烹饪时间都是10000-20000，打算在完成时间乘0.8的时候，两人都开始往回走。

烹饪完成后：
如果刚刚烹饪的是【中间产物】，和目前所有桌子上和手里的食材匹配一下，能做啥就做啥，匹配不上就回Q1。
如果刚刚烹饪的是【成品菜肴26-51】，拿起来进入**【3.手中有成品菜等待提交】**
Q4：任务列表里是否有手中的菜肴？
有：直接过去提交
无：刚刚烹饪的是【26番茄炒蛋】吗？
    是：如果桌子+灶台+手里有【鸡蛋】，就立刻接着烹饪鸡蛋面，进入**【2.有菜肴正在烹饪】**
        如果手里没有鸡蛋，同下
    否：找个靠墙的地方干等着，直到任务列表里有任务了再上交（靠墙会不会安全点啊）
    
end

### 副player
我还没想好orz
【找食材模式】
【守护模式】

### 寻路待完善的内容：
1.如果记录移动前的坐标，如果移动后和移动前的坐标相同，重新以当前坐标为基准寻路。
2.走路的时候看看面前有无trigger，如果有就把地图中这一点也设置为障碍（1），重新寻路。直到到达目的地后清除刚刚设置的障碍，恢复原始地图
3.在手上没有食材的时候，每走一格看看前面有无食材，有就捡起来并放回桌子
4.看看面前有无道具，护心镜7肥料6传送门8加速2虎头鞋1捡起来就能用，攻击性的9~15暂时不考虑。（话说虎头鞋能加速多少啊，要是不是整数又麻烦了。。）

