# 可行性分析



## 界面

使用easyx作为界面
目前PY、熊猫、池河 完成了 小猪佩奇



## 核心功能

飞机初始属性

1. 血量：默认为100点血
2. 攻击力：飞机子弹伤害 默认为10点/发 获取“专注”强化后提升至20点/发，持续5秒
3. 射速：飞机射速 默认为2发/秒 获取“狂暴”强化后提升至4发/秒，持续5秒
4. 子弹速度：飞机子弹速度 默认为10格/秒
5. 移动速度：飞机移速 默认为5格/秒 获取“加速”强化后提升至10格/秒，持续5秒



飞机移动控制

优先实现方式：用wsad控制飞机的上下左右移动

*学有余力：增加选项允许使用鼠标控制飞机的上下左右移动



关卡（随着时间自动进入更困难的关卡）的数量和形式

1. 三个阶段 通过时间自动切换

3. 怪物数据

   //共有三类小兵 1射速慢伤害低 2射速慢伤害高 3射速快伤害中
   //早期小兵1多，后期小兵2、3多
   
   //小兵血量 默认分别为100、140、180点血
   //小兵移速 默认为1格/秒、2格/秒、2格/秒
   //小兵射速 默认为1发/秒、1发/秒、2发/秒，射弹方向为飞机当前坐标
   //小兵子弹伤害 默认为10点/发、20点/发、15点/发



飞机强化：

设置强化生效判定栏位 三个栏位分别对应“加速”、“狂暴”、“专注” 0代表对应强化未生效 1代表对应强化生效

获取“加速”强化后移速提升至10格/秒，持续5秒

获取“狂暴”强化后射速提升至4发/秒，持续5秒

获取“专注”强化后子弹伤害提升至20点/发，持续5秒



BGM：选用音乐



## 额外功能

难度：随着时间推移进入不同阶段，游戏难度不断提高

背景环境：随着时间推移不断改变

BGM：音乐随着阶段切换

图鉴/收藏：展示遇见过的强化、对手、场景



## 核心功能实现

### part1 数据设计

```c++
//游戏阶段 根据时间，切换游戏阶段；进入下一阶段时，游戏难度增加，同时发放单局内飞机属性强化
int game_stage=0;
//游戏时间 游戏开始时开始计时，每次游戏结束后清零
int time=0;
//强化生效判定栏位 三个栏位分别对应“加速”、“狂暴”、“专注” 0代表对应强化未生效 1代表对应强化生效
int is_strengthen[3]={0}

//子弹 （要素：子弹位置 开火频率 子弹伤害 飞行速度 发射方向）
typedef struct bullet{
    //子弹位置
    int bullet_x,bullet_y; 
    //开火频率
    int firingrate;
    //子弹伤害
    int damage; 
    //飞行速度
    int airspeed;
    //发射方向 0：飞机所在坐标方向 1：上 2：下 3：左 4：右 5:左上 6：右上 7：左下 8：右下 9:下+左下+右下
    int direction;
}Bullet;

//飞机
struct plane{
    //飞机坐标x、y 默认位于界面下方中心处
    int plane_x,plane_y;
    //飞机血量 默认为100点血
    int plane_blood;
    //飞机移速 默认为5格/秒 获取“加速”强化后提升至10格/秒，持续5秒
    int plane_speed; 	
    //飞机射出的子弹
    Bullet plane_bullet;
};

//小兵 （共有三类 1射速慢伤害低 2射速慢伤害高 3射速快伤害中 早期小兵1多，后期小兵2、3多）
typedef struct soldier{
    //小兵位置
    int soldier_x,soldier_y; 
	//小兵血量 默认分别为100点
	int soldier_blood=100;
    //小兵移速 默认为1格/秒
	int soldier_speed=1;
    //小兵射出的子弹
    Bullet plane_bullet;
}Soldier;

//boss （500血以上时为第一形态，500血以下为第二形态，在第二形态下各项能力提升）
struct boss{
    //boss位置
    int boss_x,boss_y; 
    //boss血量 默认1000点血
    int boss_blood;
    //boss移速 第一形态为2格/秒，第二形态为3格/秒
    int boss_1_speed;
    int boss_2_speed;
    //boss射出的子弹
    Bullet boss_1_bullet;
    Bullet boss_2_bullet;
};
```

### part2 逻辑思路


```c++
//数据初始化
void Data_Initialize()
{
    //对飞机相关数据进行初始化
    
    //飞机坐标x、y 默认位于界面下方中心处
    int plane_x,plane_y;
    //飞机血量 默认为100点血
    int plane_blood=100;
    //飞机移速 默认为5格/秒 获取“加速”强化后提升至10格/秒，持续5秒
    int plane_speed=5; 
    //飞机开火频率 默认为2发/秒 获取“狂暴”强化后提升至4发/秒，持续5秒
    int plane_bullet.firingrate=2; 
    //飞机子弹伤害 默认为10点/发 获取“专注”强化后提升至20点/发，持续5秒
    int plane_bullet.damage=10; 
    //飞机子弹速度 默认为10格/秒
    int plane_bullet.airspeed=10; 
    //飞机子弹方向 向正上方
    int plane_bullet.direction=1; 
	
    //对boss相关数据进行初始化
    
    //boss血量 默认1000点血
    int boss_blood=1000;
    //boss移速 第一形态为2格/秒，第二形态为3格/秒
    int boss_1_speed=2;
    int boss_2_speed=3;
    //boss开火频率 第一形态1发/秒，第二形态2发/秒；第一形态三个射弹方向，第二形态六个射弹方向 
    int boss_1_bullet.firingrate=1;
    int boss_2_bullet.firingrate=2;
    //boss子弹伤害 
    int boss_1_bullet.damage=1;
    int boss_2_bullet.damage=2;
    //boss子弹速度
    int boss_1_bullet.airspeed=5;
    int boss_2_bullet.airspeed=5;
    //boss子弹方向 向正下方、左下方、右下方三向同时发射
	int boss_2_bullet.direction=9;
}

//控制飞机移动 飞机通过wsad键控制四个方向，在地图中移动
void Control() 
{
    if ( GetAsyncKeyState(W||w) ) {
            plane_y -= 3;
        }
        if ( GetAsyncKeyState(S||s) ) {
            plane_y += 3;
        }
        if ( GetAsyncKeyState(A||a) ) {
            plane_x -= 3;
        }
        if ( GetAsyncKeyState(D||d) ) {
            plane_x -= 3;
        }
}

//飞机状态栏
void Plain_Status_Bar()
{
	//打印血条、血量数字
	
	//打印当前数据：射速、伤害、游戏时间、游戏阶段
	
}{}
```


  	
```c++
//飞机被击中 飞机与子弹出现重合即视为被击中，子弹消失，飞机扣血
void Be_Attacked()
{
	//飞机中单判定
	if(飞机与子弹出现重合){
        子弹消失;
        if(子弹来自soldier1){
            //飞机血量-10
            plane_blood-=soldier_1_bullet_damage;
        }
        if(子弹来自soldier2){
            //飞机血量-20
            plane_blood-=soldier_2_bullet_damage;
        }
        if(子弹来自soldier3){
            //飞机血量-15
            plane_blood-=soldier_3_bullet_damage;
        }
        if(子弹来自boss){
            //飞机血量-15
            plane_blood-=soldier_1_bullet_damage;
        }
    }
    //飞机坠毁判定
    if(血量低于0点){
    	Crash();
    }
}

//飞机坠毁
void Crash()
{
	爆炸动画（爆炸图片覆盖飞机）
	绘制文字“您已被击落”
	弹出按钮返回主菜单
}

//飞机自动攻击
void Auto_Attact()
{
	//飞机按照plane_firingrate设定的频率进行射击
	
	//强化效果判定——“加速”
	if(is_strengthen[0]==1){
		plane_speed=10
		持续五秒
		plane_speed=5
	}
	//强化效果判定——“狂暴”
	if(is_strengthen[0]==1){
		plane_firingrate=4
		持续五秒
		plane_speed=2
	}
	//强化效果判定——“专注”
	if(is_strengthen[0]==1){
		plane_bullet_damage=20
		持续五秒
		plane_bullet_damage=10
	}
}

//游戏第一阶段
void First_Stage()
{
	//设定第一阶段的出怪类型和出怪时间
	
	//到设定的时间出怪
	
}

void Second_Stage()
{
	//设定第二阶段的出怪类型和出怪时间
	
	//到设定的时间出怪
	
}

void Third_Stage()
{
	//设定第三阶段的出怪类型和出怪时间
	
	//到设定的时间出怪
	
}

void Create_Background()
{
	//创建飞机
    IMAGE background;
    //加载飞机图片
    loadimage(&background, "./background.jpg", , );
}

void Create_Plane()
{
    //创建飞机
    IMAGE plane;
    //加载飞机图片
    loadimage(&plane, "./plane.jpg", 长度 , 宽度 );
}

//开始游戏
void Start_Game
{
	//开始播放BGM
	void BGM();
	while(1){
        //数据初始化
		Data_Initialize()
		//创建背景
		Create_Background();
		//创建飞机
		Create_Plane();
		
		//双缓冲绘图（需要放在绘图代码前和后）
		BeginBatchDraw();
		//放置飞机图片
		putimage(plane_x, plane_y, &cat);
		//放置状态栏
		Plain_Status_Bar();
		//控制飞机移动
		Control();
		//进行中弹判定 假如血量低于0点->损毁，游戏结束
		Be_Attacked();
		//双缓冲绘图（需要放在绘图代码前和后）
		EndBatchDraw();
		
		//游戏随时间推移切换阶段 根据阶段进行出怪
        if(time<=第二阶段开始时间){
            First_Stage();
        }
        else if(time>=第二阶段开始时间&&time<=第三阶段开始时间){
            Second_Stage();
        }
        else(time>=第三阶段开始时间){
            Third_Stage();
        }
	}
}
```


  	
  	
  	//背景音乐 BGM
  	void BGM() 
  	{
  		//打开文件
  		mciSendString("open ./XXX.mp3 alias BGM", 0, 0, 0);
  		//循环播放音乐
  		mciSendString("play BGM repeat", 0, 0, 0);
  		if (0) {
  			mciSendString("close BGM", 0, 0, 0);
  		}
  	}



  	







