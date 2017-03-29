# majiang

Go语言麻将服务器实现

Backend implementation of server in golang 

# usage

cd bin
./ctrl build
./ctrun run

# protocol
protobuf

# document

协议数据打包：
handerLen + protoLen + dataLen + data

handerLen uint32 = 1 : 包头长度1byte(包序)
protoLen uint32  = 4 : 协议号长度4(大端序)
dataLen uint32 = 4 : 数据长度4(大端序)
data uint32 = dataLen : 数据

包序算法：index++; index = index % 256

server port:7001


/*

选庄时间显示10秒，实际时间为20秒
下注时间显示10秒，实际时间为20秒
组合手牌时间显示12秒，实际时间为30秒

流程：
开始牌局 发4张牌(正面显示) 抢庄阶段(多人抢或无人抢时随机分配)

闲家下注(超时默认1倍) 发第5张牌(正面显示) 组合手牌 推开手牌

比拼大小 积分结算 结束牌局

【基本规则】
房主在创建房间时，需要选择进入人数，最少需要3名玩家后，才能开始游戏，在第一局游戏开始后，无法进入房间
总共52张牌（无大小王）
定庄后，正面展示玩家手牌，玩家可以自己根据5张牌进行排列组合，组合后，与庄家进行比拼，决定胜负
下注时，只允许闲家下注，庄家无法下注

【比拼规则】
玩家从手牌中任意选择3张进行组合（确定有牛），
组合后，点击摊牌按钮，即打开手牌
当所有玩家摊开手牌后，则与庄家比拼大小
如果是五小或炸弹牌型，则无需组合，即可选择摊牌

牌型:(牌面分数：10、J、Q、K都为10，其他按牌面数字计算)
无牛：5张中任意3张相加不能成为10的倍数
有牛：5张中任意3张相加能成为10的倍数(牛几等于另2张相加后的个位数)
牛牛：5张中任意3张相加能成为10的倍数,另外2张相加也是10的倍数
四花：5张中1张10,另外4张为花牌(大于10的牌)
五花：5张全部为花牌
五小：5张全部小于5且相加小于10
炸弹：5张中4张相同的牌

大小比较
牌型：炸弹>五小>五花>四花>牛牛>有牛>无牛
花色：黑桃>红桃>草花>方块
单张：K>Q>J>10>9>8>7>6>5>4>3>2>A
无牛：比单张大小
有牛：比牌型大小，牛九>牛八>牛七>牛六>牛五>牛四>牛三>牛二>牛一
牛牛：比单张+花色的大小
四花：比单张+花色的大小
五花：比单张+花色的大小
五小：比点数+单张+花色的大小
炸弹：大牌吃小牌，K最大，A最小

积分计算
无牛,牛1~6:   1倍
牛7~9:        2倍
牛牛：        3倍
四花：        4倍
五花：        5倍
五小：        6倍
炸弹：        6倍

玩家胜负积分=倍数*抢庄下注额度
如：玩家A抢庄成为庄家，玩家B下注4，以牛牛牌型获得胜利，
则，玩家A需支付给玩家B的分数为： 4*3=12

*/

const (
	HgihCard = iota + 0x00
	Niu1
	Niu2
	Niu3
	Niu4
	Niu5
	Niu6
	Niu7
	Niu8
	Niu9
	NiuNiu
	FourFlower
	FiveFlower
	FiveTiny
	Bomb
)

// rank
const (
	Ace = iota + 0x01
	Deuce
	Trey
	Four
	Five
	Six
	Seven
	Eight
	Nine
	Ten
	Jack
	Queen
	King

	RankMask = 0x0F
	RemMask  = 0x0A
)

// suit
const (
	Club    = 0x40
	Heart   = 0x30
	Spade   = 0x20
	Diamond = 0x10

	SuitMask = 0xF0
)

const (
	NumCard = 52
)
