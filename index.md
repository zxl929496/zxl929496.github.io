#### 导语

> 这是一款人机对弈的象棋游戏。它以简单的棋盘棋子作为整个界面，布局简单清晰。用户执黑棋先行，机器执红棋后行，用户只需要单击鼠标就可以控制棋子的移动，让你不仅仅是娱乐，还可以不断提高棋艺。目前已经在AppStore上架，主要针对iphone手机用户，可以搜索“**象棋宝宝**”免费下载。
## 项目背景
学习iOS开发以来，"象棋宝宝"是我的第一个项目。一路过来，遇到了很多的难题，很多的第一次；可以说是在不断地解决难题，不断进步的过程中，这个项目才能有现在的成果；当然了，为了让它变得更好，还需要不断地优化。
## 开发过程
### 一 界面
这里我没有使用Automaticlayout来实现界面，是通过代码来实现的。基于面相对象的思想，我创建了一个Checkerboard.swift的类文件，生成了棋盘图片vImg、棋盘在ViewController的原点位置tlx、tly、棋盘的宽高width、height等属性，用init（）构造函数来初始化这些属性，在ViewController.swift里形成一个棋盘的实例变量CheckB，在viewDidload里加载棋盘图片。
```
class Checkerboard
{
    private var vImg: UIImageView
    private var tlx, tly, width, height: Int
    init(name: String, x: Int, y: Int, wd: Int, hg: Int)
    {
        let img = UIImage(named: name)
        vImg = UIImageView(image: img)
        vImg.frame = CGRect(x: x, y: y, width: wd, height: hg)//根据棋盘的原点x、y、width、height的像素点，用CGRect来确定位置
        tlx = x
        tly = y
        width = wd
        height = hg
    }
}
class ViewController:UIViewController
{
    static var checkB = Checkerboard(name: "timg1", x: 10, y: 60, wd: Int(UIScreen.main.bounds.size.width - 20), hg: Int(UIScreen.main.bounds.size.height - 120))  //UIScreen.main.bound获取整个屏幕的width,height.
    override func viewDidLoad() {
        super.viewDidLoad()
        self.view.addSubview(ViewController.checkB.getvImg())
    }//加载棋盘Imageview到view上
}
```
现在ViewController界面已经有了棋盘，我用同样的方法创建了Piece.swift文件，生成了棋子的颜色color、在棋盘的行列row、column、最初的行列startrow、startcolumn、图片imgv、威力值pow、灵活度权值flwt等，用init()构造函数来初始化每一个棋子；在ViewController.swift文件里生成了所有棋子初始位置的pieces数组。我的图片都是通过在Assets.xcassets点击右键import本地文件。现在，我的棋盘棋子已经在最初始的位置了。
```
class Piece: NSObject
{
    private var color: Bool
    private var row: Int
    private var column: Int
    private var startrow: Int
    private var startcolumn: Int
    private var imgv: UIImageView
    private var pow: Int
    private var flwt: Int
    private var xorigin: Int
    {
        return ViewController.getcheckB().getcspace() * column  //通过棋盘的行列间距、棋子的行列数算出每一个棋子在棋盘上的原点坐标
    }
    private var yorigin: Int
    {
        return ViewController.getcheckB().getrspace() * row - ViewController.getcheckB().getcspace() / 2 + ViewController.getcheckB().getrspace() / 2
    }
    init(col: Bool, r: Int, c: Int, sr: Int, sc: Int, name: String, power: Int, flbywt: Int)
    {
        color = col
        row = r
        column = c
        startrow = sr
        startcolumn = sc
        pow = power
        flwt = flbywt
        let imag = UIImage(named: name)
        imgv = UIImageView(image: imag)//ImageView加载图片
        super.init()
        imgv.frame = CGRect(x: xorigin, y: yorigin, width: ViewController.getcheckB().getcspace(), height: ViewController.getcheckB().getcspace())//
    }
    class ViewController: UIViewController
    {
        static var pieces = [Car(col: true, r: 0, c: 0, sr: 0, sc: 0, name: "Rju", power: 800, flbywt: 8),
                         Horse(col: true, r: 0, c: 1, sr: 0, sc: 1, name: "Rma", power: 300, flbywt: 10),
                         Elephant(col: true, r: 0, c: 2, sr: 0, sc: 2, name: "Rxiang", power: 300, flbywt: 6)]////这只是一部分棋子，依次，所有的棋子都在最开始的棋盘位置。
        static var checkbd = [[Piece?]](repeating: [Piece?](repeating: nil, count: 9), count: 10)//通过二维数组checkbd可以知道棋盘上每一行列是空的还是有哪一个棋子
        func addTouchPieceView()
    {
        for piece in ViewController.pieces
        {
            ViewController.checkB.getvImg().addSubview(piece.getimgv())
        }//把棋子Imageview一一添加到棋盘view上
    } 
```
### 二添加点击响应事件使棋子移动
怎样才能让用户点击棋子的起点和终点来移动棋子呢？在这里，我添加了两个点击事件：棋盘view点击事件、棋子view点击事件；当然了，象棋不是随便移动的，得要按照每个棋子的规则来。我给每一种的棋子建立一个单独的.swift类文件，继承棋子Piece这个类分别知道棋子在当前位置可以走的所有终点位置。
```
class Horse: Piece//马
{
    //是否马能移动
    func canHorseMove(cr: Int, cco: Int) -> Bool
    {
        let a = abs(getrow() - cr), b = abs(getcolumn() - cco)
        let dmr = (cr - getrow()) / 2 + getrow(), dmc = (cco - getcolumn()) / 2 + getcolumn()
        if ((a == 2 && b == 1) || (a == 1 && b == 2))
            && (ViewController.checkbd[dmr][dmc] == nil)
        {
            return true
        }
        return false
    }
    
    //重写父类move函数 马怎么走
    override func move(cr: Int, cco: Int) -> Bool
    {
        if canHorseMove(cr: cr, cco: cco) == true
        {
            domove(cr: cr, cco: cco)
            return true
        }
        return false
    }
    
    //重写父类allCanMovePlaces函数 step类型是一个包括棋子起点sx、sy和终点ex、ey的结构体 函数输出某一位置的马所有能移动位置的的step类型
    override func allCanMovePlaces() -> [step]
    {
        let sx = getrow(), sy = getcolumn()
        var places = [step]()// 马按规则所有的位置数组
        var mvplaces = [step]()//马在棋盘位置所有能走的数组
        places.append(step(sx: sx, sy: sy, ex: sx - 1, ey: sy - 2))
        places.append(step(sx: sx, sy: sy, ex: sx - 1, ey: sy + 2))
        places.append(step(sx: sx, sy: sy, ex: sx - 2, ey: sy - 1))
        places.append(step(sx: sx, sy: sy, ex: sx - 2, ey: sy + 1))
        places.append(step(sx: sx, sy: sy, ex: sx + 1, ey: sy - 2))
        places.append(step(sx: sx, sy: sy, ex: sx + 1, ey: sy + 2))
        places.append(step(sx: sx, sy: sy, ex: sx + 2, ey: sy - 1))
        places.append(step(sx: sx, sy: sy, ex: sx + 2, ey: sy + 1))
        for i in 0..<places.count
        {
            if places[i].ex <= 9 && places[i].ex >= 0
                && places[i].ey >= 0 && places[i].ey <= 8
                && canHorseMove(cr: places[i].ex, cco: places[i].ey) == true
                && getcolor() != ViewController.checkbd[places[i].ex][places[i].ey]?.getcolor()
            {
                mvplaces.append(places[i])
            }
        }
        return mvplaces
    }
}
```
```
class viewController: UIViewController
{
    func addTouchCheckbView()
    {
        ViewController.checkB.getvImg().isUserInteractionEnabled = true//打开用户交互界面 棋盘和棋子的都要打开
        let cbsingleTap: UITapGestureRecognizer = UITapGestureRecognizer(target: self, action: #selector(ViewController.touchcb))//UITapGestureRecognizer类是一个处理单机事件的,响应touchcb()函数
        cbsingleTap.numberOfTapsRequired = 1 //点击一下
        ViewController.checkB.getvImg().addGestureRecognizer(cbsingleTap) //棋盘checkB添加手势识别器
    }
    func addTouchPieceView()
    {
        for piece in ViewController.pieces
        {
            ViewController.checkB.getvImg().addSubview(piece.getimgv())
            piece.getimgv().isUserInteractionEnabled = true //打开用户交互界面
            let pcsingleTap: UITapGestureRecognizer = UITapGestureRecognizer(target: self, action: #selector(ViewController.touchPiece))
            pcsingleTap.numberOfTapsRequired = 1
            piece.getimgv().addGestureRecognizer(pcsingleTap)
            ViewController.checkbd[piece.getrow()][piece.getcolumn()] = piece
        }
    }
}
```
棋盘棋子点击后是怎么响应的呢？这里规定红棋是机器自动走，黑棋是用户点击走动，每次都是黑棋先行。所以，只有黑棋走才会点击响应事件，然后判断是否走；黑棋移动后，机器才走；不断地重复，直到一方被**将军**，跳出警报窗口，这盘棋才结束。
```
class ViewController: UIViewController
{
    private var firsttap = false//用来判断第一次点击的是棋盘（false）还是棋子(true)
    private var isBlackMove = true//判断目前是黑棋走（true）还是红棋（false）走
    
    //棋盘点击只有一种情况：点击终点走棋
    func touchcb(sender: UITapGestureRecognizer)
    {
        if !isBlackMove
        {
            return //红棋走不需要做什么
        }
        if firsttap == true
        {
            let point = sender.location(in: ViewController.checkB.getvImg())//获取点击位置的原点坐标
            let cbrow = (Int(point.y) - 25 + ViewController.checkB.getcspace() / 2) / ViewController.checkB.getrspace()
            let cbcolumn = (Int(point.x) - 35 + ViewController.checkB.getrspace() / 2) / ViewController.checkB.getcspace()
            
            //判断终点是否能走棋
            if (ViewController.checkbd[cbrow][cbcolumn] != nil
                && ViewController.checkbd[cbrow][cbcolumn]!.getcolor() != ViewController.checkbd[prow][pcolumn]!.getcolor())
                || ViewController.checkbd[cbrow][cbcolumn] == nil
            {
                let cdmove = ViewController.checkbd[prow][pcolumn]!.move(cr: cbrow, cco: cbcolumn)//判断棋子从起点到终点是否按规则走
                isBlackMove = !cdmove//true:  下一步红棋走,false: 黑棋走
                if cdmove == true && ViewController.checkB.getWiner() != 0
                {
                    DispatchQueue.global().async {
                        self.mechineMove() // 棋局没结束，机器走
                    }
                }
                if ViewController.checkB.getGameOver() == true
                {
                    gameAlert()//弹出警报窗口
                    ViewController.checkB.setGameOver(gameov: false)
                }
            }
            firsttap = false
        }
    }
    
    //棋子 两种情况：起点点击、终点点击
    func touchPiece(sender: UITapGestureRecognizer)
    {
        if !isBlackMove
        {
            return
        }
        let point = sender.location(in: ViewController.checkB.getvImg())
        if firsttap == false
        {
            prow = (Int(point.y) - 25 + ViewController.checkB.getcspace() / 2) / ViewController.checkB.getrspace()
            pcolumn = (Int(point.x) - 35 + ViewController.checkB.getrspace() / 2) / ViewController.checkB.getcspace()
            if ViewController.checkbd[prow][pcolumn]?.getcolor() == false
            {
                firsttap = true
            }
        }
        else
        {
            let tpr = (Int(point.y) - 25 + ViewController.checkB.getcspace() / 2) / ViewController.checkB.getrspace()
            let tpc = (Int(point.x) - 35 + ViewController.checkB.getrspace() / 2) / ViewController.checkB.getcspace()
            if ViewController.checkbd[prow][pcolumn]!.getcolor() != ViewController.checkbd[tpr][tpc]!.getcolor()
            {
               let dmove = ViewController.checkbd[prow][pcolumn]!.move(cr: tpr,cco: tpc)
               isBlackMove = !dmove
                if dmove == true && ViewController.checkB.getWiner() != 0
                {
                    DispatchQueue.global().async {
                        self.mechineMove()//机器走
                    }
                }
                if ViewController.checkB.getGameOver() == true
                {
                    gameAlert()
                    ViewController.checkB.setGameOver(gameov: false)
                }
            }
            firsttap = false
        }
    }
}
```
现在我点击响应都有了，怎么让用户和机器走棋呢？我
这里每一个棋子都有自己的类，来判断是否符合规则move，但是，在用户界面上是否能真的移动,怎么移动呢？用户（黑棋）走棋有两种情况，一是终点位置是空的，直接走；二是终点位置有对方的棋，"吃子,"吃子"还有一种特殊的情况，就是**将军**，棋局结束。这里开始我遇到的问题是：我更新棋子imgv的时候，用户体验到的是棋子直接从起点到终点，没有任何过程，这样效果就不行了。我现在是给棋子移动更新添加了一个动画过程：用户可以看到棋子是从起点沿着棋盘方格对角线有轨迹的移过去。
```
class Pieces: NSObject
{
    func domove(cr: Int, cco: Int)
    {
        //吃子
        if ViewController.checkbd[cr][cco] != nil
        {
            let tmp = ViewController.checkbd[cr][cco];
            ViewController.checkbd[cr][cco]!.getimgv().removeFromSuperview() //调用removeFromSuperview()函数，将被吃掉的棋子view从棋盘view移掉
            for i in 0..<ViewController.pieces.count
            {
                var t: Piece?
                t = ViewController.pieces[i]
                if tmp == t
                {
                    ViewController.pieces.remove(at: i)
                    break//被吃掉的棋子从pieces数组移掉
                }
            }
        }
        //“将军” 10000的威力值是将军和帅
        if ViewController.checkbd[cr][cco]?.getpow() == 10000
        {
            ViewController.checkB.setGameOver(gameov: true) //判断游戏结束
            if ViewController.checkbd[cr][cco]?.color == true //判断谁输谁赢
            {
                ViewController.checkB.setWiner(win: 0)
            }
            else
            {
                ViewController.checkB.setWiner(win: 1)
            }
        }
        //棋子移动位置，棋子属性row、column和imgv的origin也要变
        ViewController.checkbd[cr][cco] = nil
        ViewController.checkbd[cr][cco] = ViewController.checkbd[row][column]
        ViewController.checkbd[row][column] = nil
        ViewController.checkbd[cr][cco]?.setrow(cbrow: cr)
        ViewController.checkbd[cr][cco]?.setcolumn(cbcolumn: cco)
        ViewController.checkbd[cr][cco]?.setimgv()//调用playanimation()实现移动更新UI效果
    }
}
```
```
func playanimation()
    {
        UIImageView.animate(withDuration: 1, delay: 0, options: [.beginFromCurrentState],
                            animations:{
                                self.imgv.frame.origin.x = CGFloat(self.xorigin)
                                self.imgv.frame.origin.y = CGFloat(self.yorigin)
        },
                            completion: nil)
    }
func setimgv()
{
    playanimation()
}
```
机器（红棋）是怎么自动实现走棋的呢？第一，机器是怎么知道选出最佳的一步来走：这里我是通过试走红棋所有可以移动的棋子，计算棋面的总的分数来决定的。单个棋的分数是棋的灵活度权值与棋的可移动次数乘积加上棋的威力值；整个棋面的score值是：score + 红棋 - 黑棋。

```
var score = 0
for j in 0..<ViewController.pieces.count
        {
            let numb = ViewController.pieces[j].allCanMovePlaces().count//每个棋子的可移动次数
            if ViewController.pieces[j].getcolor() == true
            {//红棋 加
                score += ViewController.pieces[j].getpow() + ViewController.pieces[j].getFlwt() * numb
            }
            else//黑棋 减
            {
                score -= (ViewController.pieces[j].getpow() + ViewController.pieces[j].getFlwt() * numb)
            }
        }

```
第二，“走一步看三步”，怎么做到呢？先看看走一步是怎么搜索的。假设现在用户（黑棋）走了一步，机器（红棋）应该走哪一步是最佳的呢？机器将所有can move的都试走一遍，但不在界面反应，记录每走一步后形成的局面分数，棋子再还原,最后比较出分数最高的作为最佳实际走的选择。走两步呢，这里我用的是极大值极小值算法：max（红棋取最大值）、min（黑棋取最小值）代表对弈双方，当轮到红棋走时，应该考虑到最好的情况，局面分数取最大；当轮到黑棋走时，应该考虑最坏的情况，局面分数取最小；交替使用这两种情况传递倒推值，使用递归的方法。每次在叶子节点中每个叶子节点形成的局面计算每个叶子节点的分值，如果叶子节点的跟节点是红棋，那么返回最大值；黑棋返回最小值；依次递归向上。但是，当搜索层数越多，节点就越多，时间要的多，棋子反应速度就降下去了，所以，需要比较一些不需要再搜索下去的节点删掉，需要用到极大值极小值的剪枝。假设搜两层，第一层红棋找最大值返回根节点；第二层是黑棋，由第一层的每一个节点作为根节点，每一个根节点下的所有子叶子节点组成，每一根节点下的叶子节点找最小值返回根节点第一层；如果我们知道了第二层第一个返回根节点第一层的值，那么，是不是表示，第一层其他根节点的叶子节点只要有分值比返回值小，返回给根节点的一定小于或等于它，而在第一层却要返回最大值给第一层的根节点时一定会被淘汰掉，所以，可以直接把叶子节点和它所在的根节点直接剪枝，不需要计算。

```
func dfs(level: Int, lastlevel: Int, bs: Int) -> Int
    {
        if level == lastlevel
        {
            return calculateSorce()//叶子节点计算分值
        }
        else
        {
            var result = level % 2 == 0 ? Int.min : Int.max
            let steps = allSteps(level: level)//当前一方所有能走的步数
            for step in steps
            {
                let start = ViewController.checkbd[step.sx][step.sy]
                let end  = ViewController.checkbd[step.ex][step.ey]
                start!.tryMove(cr: step.ex, cco: step.ey)//试走
                let score = dfs(level: level + 1, lastlevel: lastlevel, bs: result)//算分只值
                start!.revert(pr: step.ex, pco: step.ey, cr: step.sx, cco: step.sy, tps: end)//还原
                
                if level % 2 == 0  //max
                {
                    if(score > result)
                    {
                        result = score
                        bestmoves = (level == 0 ? step : bestmoves)//更新最佳  bestmoves是最佳走法的起点终点step类型
                    }
                    if score > bs//剪枝
                    {
                        break
                    }
                }
                else if level % 2 == 1  //min
                {
                    result = score < result ? score: result
                    if score < bs
                    {
                        break
                    }
                }
            }
            return result
        }
    }

```
机器现在能走了，我最开始就是直接在主线程里面执行它，呈现出的效果是：用户点击终点后，界面上用户的棋和机器同时更新走动；不是我想要的，用户与机器是有先后顺序的，用户走完了，机器思考再走；后来我找到原因了，原来，UI界面是在主线程执行完了才开始同时更新的，才会有最开始的那种效果。现在，我用了多线程，机器让它在另外一个线程上执行


```
 DispatchQueue.global().async   //后台线程计算机器最佳走法
 {
     self.mechineMove()
 func mechineMove() 
 {
    let _ = dfs(level: 0, lastlevel: 3, bs: Int.max)
        DispatchQueue.main.async //返回主线程更新UI{
        ViewController.checkbd[self.bestmoves.sx][self.bestmoves.sy]!.domove(cr: self.bestmoves.ex, cco: self.bestmoves.ey)
            self.isBlackMove = true
            if ViewController.checkB.getGameOver() == true
            {
                self.gameAlert()
                ViewController.checkB.setGameOver(gameov: false)
            }
        
        }
```
### 三 动画警告框
不管哪一方被**将军**，都表示这局结束了，要怎样继续重新来一局呢？这里我通过在touchcb() 和touchPiece() 调用gameAlert（）弹出一个警告框，用户点击“**重来**”，棋面重新恢复。棋面怎么恢复到最初的状态呢？很简单，我只需要确定棋盘view上是没有棋子的；棋子view的每一个棋子在最开始的行列上；
```
func gameAlert()
    {
        var sucesser = ""
        if ViewController.checkB.getWiner() == 0
        {
            sucesser = "黑棋赢了"
            ViewController.checkB.setWiner(win: 2)
        }
        else
        {
            sucesser = "红棋赢了"
        }
        let alertController = UIAlertController(title: "系统提示"                           message: sucesser, preferredStyle: .alert)
        let okAction = UIAlertAction(title: "重来", style: .default, handler: {
            action in
            self.presentingViewController?.dismiss(animated: true, completion: nil)
            for i in 0..<10
            {
                for j in 0..<9
                {
                    // 恢复棋盘view
                    if ViewController.checkbd[i][j] != nil
                    {
                       ViewController.checkbd[i][j]?.getimgv().removeFromSuperview()
                       ViewController.checkbd[i][j] = nil
                    }
                }
            }
        
            //恢复每一个棋子view 
            for piece in ViewController.startpieces
            {
                piece.setrow(cbrow: piece.getStartrow()) //在pieces数组里startrow、startcolumn属性 记录每个棋子最开始的行列
                piece.setcolumn(cbcolumn: piece.getStartcolumn())
                piece.setimgv()
                ViewController.checkbd[piece.getStartrow()][piece.getStartcolumn()] = piece
                ViewController.checkB.getvImg().addSubview(piece.getimgv())//行列一定不能忘了改变棋子view
                ViewController.pieces = ViewController.startpieces
            }
        })
        alertController.addAction(okAction)//添加动画
        self.present(alertController, animated: true, completion: nil)//显示提示框
    }
```

### 四 “新局”
当你玩到中途想重来一局时，只需要点击屏幕下方的button就可以了。我添加了一个“新局”button，点击以后会跳出警告框，点击确定就可以重来，‘点击取消不会改变什么。我在button里添加了一个addTarget函数来响应到
Alert函数弹出警告框。

```
 func newBoardButton()
{
    butt.addTarget(self, action: #selector(ViewController.Alert), for: .touchUpInside)
}
func Alert()
    {
        let alertController = UIAlertController(title: "新局", message: nil, preferredStyle: .alert)
        let okAction = UIAlertAction(title: "确定", style: .default, handler: {
            action in
            code //代码和将军后重来是一样的
            })
    let cancelAction = UIAlertAction(title: "取消", style: .cancel, handler: nil)
        alertController.addAction(okAction)
        alertController.addAction(cancelAction)
        self.present(alertController, animated: true, completion: nil)
    }
```
### 五 为了用户在游戏中不会那么的单调，我添加了工程背景音乐，在函数playMusic()导入MP3格式的音乐。在函数voiceButton()添加一个button，有两种状态：播放、停止；
```
import UIKit
import AVFoundation  //引入新的AVFoundation框架
class ViewController: UIViewController 
{
    var audioplayer= AVAudioPlayer() //定义AVAudioPlayer类型的全局变量并初始化
    var button: UIButton
    func playMusic()
    {
        let musicPath = Bundle.main.path(forResource: "于秋旋-高山流水", ofType: 			"mp3")
        let url = NSURL(fileURLWithPath: musicPath!)
//指定音乐入径 URL
       	do
//错误处理
        {
            let sound = try AVAudioPlayer(contentsOf: url as URL)
    		//确保audioplayer有值
            audioplayer = sound
            audioplayer.numberOfLoops = -1// 设置播放次数 -1 为循环
            audioplayer.volume = 0.5// 设置声音 范围 0 - 1
            audioplayer.prepareToPlay()
           	audioplayer.play()// 准备好播放
       	}
        catch
    	{
    	    print(error)
       	}
    }
    func voiceButton()
// 手动添加控制音乐的点击按钮
    {
        button = UIButton(frame: CGRect(x: UIScreen.main.bounds.width / 2 - 20, y: UIScreen.main.bounds.height / 2 - 50, width: 50, height: 50))//设置button的位置
        let bimg = UIImage(named: "timgb")
    	button.setImage(bimg, for: UIControlState.normal)//初始化button	
   	 	button.addTarget(self, action:#selector(ViewController.voice), for: .touchUpInside)//添加点击事件 voice
       		 self.view.addSubview(button)// 将button添加到view上
   	 }
    func voice()
    {
        if !audioplayer.isPlaying
        { //停止状态-点击-播放
            audioplayer.play()
            button.setImage(UIImage(named:"timgb"),for:.normal)
        }
        else
       	{ //播放-点击-停止
            audioplayer.pause()
       	    button.setImage(UIImage(named:"timgz"),for:.normal)
       	}
    }
    override func viewDidLode()
    {
        self.playMusic()
        self.voiceButton
    }	
}
```
#### 总结
目前，“象棋宝宝”也就简单地实现了这几个基本功能，还需要不断完善，让功能更加丰富，提高用户体验。
