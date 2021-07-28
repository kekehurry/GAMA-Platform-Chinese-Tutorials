# Step3: 父、子族群的继承

在我们的模型中食草动物和食肉动物有几乎一样的属性和行为，只是两者吃的东西不一样，比起重新编写一次食肉动物的行为，利用父族群和子族群的继承关系，避免重复的代码是更高效的写法。

### 父、子族群

在GAML中子族群可以继承族群所有的属性和行为，我们可以定义一个通用族设为父族，将食肉动物和食草动物共有的属性定义在父族中，并让他们从父族继承这些共有属性，而将他们不同的属性分别定义在各自族群里。

![4.3.1 &#x7236;&#x65CF;&#x7FA4;&#x548C;&#x5B50;&#x65CF;&#x7FA4;](../../.gitbook/assets/image%20%285%29.png)

### 共有属性

* 共有变量：
  * 大小：                        size
  * 颜色：                        color
  * 所在网格:                    my\_cell
  * 最大能量值：            max\_energy
  * 每次最大进食量：    max\_transfer
  * 每次能量消耗量：    energy\_consum
  * 所在网格:                    my\_cell
  * 能量值:                        energy
  * 繁殖概率:                    proba\_reproduce
  * 最大繁殖数：             nb\_max\_offsprings
  * 繁殖能量:                    energy\_reproduce                  
* 共有行为：
  * 移动：                         basic\_move
  * 进食：                         eat
  * 死亡：                         die
  * 繁殖：                         reproduce
* 共有显示：
  * 显示方式：                 base

注意食草动物和食肉动物都有吃的行为，但是两者具体操作稍有差异，因此我们在父族中实现一个函数energy\_from\_eat，并在子族中重写这个函数，子族重写的函数会覆盖父族同样的函数，从而实现食草动物和食肉动物不同的吃的行为。

### 父族代码实现

```text
//创建通用族
species generic_species {
	//定义属性
	//显示大小
	float size;
	//颜色
	rgb color;
	//所在的草地网格
	vegetation_cell my_cell;
	//最大能量值
	float max_energy;
	//每次进食量
	float max_transfert;
	//每次能量消耗值
	float energy_consum;
	//初始化能量为（0-max_energy)之间的随机数，每次模拟消耗energy_consum的能量，能量的最大值为max_energy
	float energy <- rnd(max_energy) update: energy - energy_consum max: max_energy;
	//繁殖概率
	float proba_reproduce;
	//最大繁殖数
	int nb_max_offsprings;
	//繁殖能量
	float energy_reproduce;
	
	//初始化位置为所在草地网格位置
	init {
		location <- my_cell.location;
	}
	
	//定义移动行为
	reflex basic_move {
		//从当前所在网格的周边相距为2的网格中选一个设置为当前网格
		my_cell <- one_of(my_cell.neighbors2);
		//改变位置至当前网格的位置
		location <- my_cell.location;
	}
	
	//定义进食行为,子族群进食行为不一样，通过重写energy_from_eat实现不一样的进食行为
	reflex eat {
		energy <- energy + energy_from_eat();
	}
	float energy_from_eat {
    return 0.0;
    }
	
	//定义死亡行为，当能量小于0时，死亡
	reflex die when: energy <= 0 {
		do die;
	}
	
	//当能量大于繁殖能量，并且繁殖概率为真时，进行繁殖行为
	reflex reproduce when: (energy >= energy_reproduce) and (flip(proba_reproduce)) {
		//繁殖数量为1-最大繁殖数之间的随机数
		int nb_offsprings <- rnd(1, nb_max_offsprings);
		//生成nb_offsprings个与自身相同族群的代理
		create species(self) number: nb_offsprings {
			//新代理网格为当前网格
			my_cell <- myself.my_cell;
			//新代理位置为当前位置
			location <- my_cell.location;
			//新代理能量为当前能量除以繁殖数
			energy <- myself.energy / nb_offsprings;
		}
		//自身能量变为当前能量除以繁殖数
		energy <- energy / nb_offsprings;
	}
	
	//显示方式：大小为size的圆，颜色为color
	aspect base {
		draw circle(size) color: color;
	}
}
```

注意父族在编写时只是创建了相关属性，并没有初始化属性值，这些属性值将在子族中初始化，以实现子族不同的属性。

### 食草动物族\(prey species\)

食草动物族继承自通用族（generic\_species\)，在其族定义中需要初始化各属性值，并重写energy\_from\_eat函数，实现其与食肉动物不同的属性与吃的行为。

```text
//创建食草动物族，其父族为generic_species 
species prey parent: generic_species {
    //初始化显示大小
    float size <- 1.0;
    //初始化颜色
    rgb color <- #blue; 
    //初始化最大能量值
    float max_energy <- prey_max_energy ;
    //初始化每次最大进食量
    float max_transfert <- prey_max_transfert ;
    //初始化每次能量消耗值
    float energy_consum <- prey_energy_consum ;
    //初始化繁殖概率
    float proba_reproduce <- prey_proba_reproduce ;
    //初始化最大繁殖数
    int nb_max_offsprings <- prey_nb_max_offsprings ;
    //初始化繁殖能量
    float energy_reproduce <- prey_energy_reproduce ;
    //重写energy_from_eat函数
    float energy_from_eat {
    //初始化能量转移量
    float energy_transfert <- 0.0;
    //当所在网格的食物量大于0时
    if(my_cell.food > 0) {
        //能量转移量为每次最大进食量与网格内食物量的最小值
        energy_transfert <- min([max_transfert, my_cell.food]);
        //更新网格内食物量
        my_cell.food <- my_cell.food - energy_transfert;
    } 
    //返回能量转移量         
    return energy_transfert;
    }
}
```

### 食肉动物群

同样地，食肉动物族继承自通用族（generic\_species\)，在其族定义中需要初始化各属性值，并重写energy\_from\_eat函数，实现其与食草动物不同的属性与吃的行为。应该注意的是，食肉动物吃了食草动物之后代表食草动物死亡，这里我们通过**ask函数**实现不同族之间的操作，具体代码如下：

```text
//创建食肉动物族，其父族为generic_species 
species predator parent: generic_species {
    //初始化显示大小
    float size <- 1.0;
    //初始化颜色
    rgb color <- #blue; 
    //初始化最大能量值
    float max_energy <- predator_max_energy ;
    //初始化每次最大进食量
    float max_transfert <- predator_max_transfert ;
    //初始化每次能量消耗值
    float energy_consum <- predator_energy_consum ;
    //初始化繁殖概率
    float proba_reproduce <- predator_proba_reproduce ;
    //初始化最大繁殖数
    int nb_max_offsprings <- predator_nb_max_offsprings ;
    //初始化繁殖能量
    float energy_reproduce <- predator_energy_reproduce ;
    //重写energy_from_eat函数
    float energy_from_eat {
    //初始化食肉动物能量转移量
    float energy_transfert <- predator_energy_transfert ;
    //列出所在网格内的食草动物
    list<prey> reachable_preys <- prey inside (my_cell); 
    //如果食草动物列表不为空   
    if(! empty(reachable_preys)) {
        //随机吃掉一个食草动物，那个食草动物死去
        ask one_of (reachable_preys) {
        do die;
        }
        //返回食肉动物每次进食量
        return energy_transfert;
    }
    //如果食草动物列表为空，返回0
    return 0.0;
    }
}
```

> **ask species/agent { do something }**: ask函数通常用来在一个族的代理运行过程中与另一个族的代理进行交互，和英语的ask sb do something类似。

### 更新全局定义和实验设置

加入肉食动物后，全局设置和实验设置中的参数，也要对应加上，本节完整代码如下：

```text
model prey_predator

global {
	// 定义食草动物的全局参数
	int nb_preys_init <- 200;
	float prey_max_energy <- 1.0;
	float prey_max_transfert <- 0.1;
	float prey_energy_consum <- 0.05;
	float prey_proba_reproduce <- 0.01;
	int prey_nb_max_offsprings <- 5;
	float prey_energy_reproduce <- 0.5;
	// 定义食肉动物的全局参数
	int nb_predators_init <- 20;
	float predator_max_energy <- 1.0;
	float predator_energy_transfert <- 0.5;
	float predator_energy_consum <- 0.02;
	float predator_proba_reproduce <- 0.01;
	int predator_nb_max_offsprings <- 3;
	float predator_energy_reproduce <- 0.5;
	//初始化
	init {
		//创建食草动物数量为nb_preys_init
		create prey number: nb_preys_init;
		//创建食肉动物数量为nb_preys_init
		create prey number: nb_preys_init;
	}
}

//创建通用族
species generic_species {
	//定义属性
	//显示大小
	float size;
	//颜色
	rgb color;
	//所在的草地网格
	vegetation_cell my_cell;
	//最大能量值
	float max_energy;
	//每次进食量
	float max_transfert;
	//每次能量消耗值
	float energy_consum;
	//初始化能量为（0-max_energy)之间的随机数，每次模拟消耗energy_consum的能量，能量的最大值为max_energy
	float energy <- rnd(max_energy) update: energy - energy_consum max: max_energy;
	//繁殖概率
	float proba_reproduce;
	//最大繁殖数
	int nb_max_offsprings;
	//繁殖能量
	float energy_reproduce;
	
	//初始化位置为所在草地网格位置
	init {
		location <- my_cell.location;
	}
	
	//定义移动行为
	reflex basic_move {
		//从当前所在网格的周边相距为2的网格中选一个设置为当前网格
		my_cell <- one_of(my_cell.neighbors2);
		//改变位置至当前网格的位置
		location <- my_cell.location;
	}
	
	//定义进食行为,子族群进食行为不一样，通过重写energy_from_eat实现不一样的进食行为
	reflex eat {
		energy <- energy + energy_from_eat();
	}
	float energy_from_eat {
    return 0.0;
    }
	
	//定义死亡行为，当能量小于0时，死亡
	reflex die when: energy <= 0 {
		do die;
	}
	
	//当能量大于繁殖能量，并且繁殖概率为真时，进行繁殖行为
	reflex reproduce when: (energy >= energy_reproduce) and (flip(proba_reproduce)) {
		//繁殖数量为1-最大繁殖数之间的随机数
		int nb_offsprings <- rnd(1, nb_max_offsprings);
		//生成nb_offsprings个与自身相同族群的代理
		create species(self) number: nb_offsprings {
			//新代理网格为当前网格
			my_cell <- myself.my_cell;
			//新代理位置为当前位置
			location <- my_cell.location;
			//新代理能量为当前能量除以繁殖数
			energy <- myself.energy / nb_offsprings;
		}
		//自身能量变为当前能量除以繁殖数
		energy <- energy / nb_offsprings;
	}
	
	//显示方式：大小为size的圆，颜色为color
	aspect base {
		draw circle(size) color: color;
	}
}

//创建食草动物族，其父族为generic_species 
species prey parent: generic_species {
    //初始化显示大小
    float size <- 1.0;
    //初始化颜色
    rgb color <- #blue; 
    //初始化最大能量值
    float max_energy <- prey_max_energy ;
    //初始化每次最大进食量
    float max_transfert <- prey_max_transfert ;
    //初始化每次能量消耗值
    float energy_consum <- prey_energy_consum ;
    //初始化繁殖概率
    float proba_reproduce <- prey_proba_reproduce ;
    //初始化最大繁殖数
    int nb_max_offsprings <- prey_nb_max_offsprings ;
    //初始化繁殖能量
    float energy_reproduce <- prey_energy_reproduce ;
    //重写energy_from_eat函数
    float energy_from_eat {
    //初始化能量转移量
    float energy_transfert <- 0.0;
    //当所在网格的食物量大于0时
    if(my_cell.food > 0) {
        //能量转移量为每次最大进食量与网格内食物量的最小值
        energy_transfert <- min([max_transfert, my_cell.food]);
        //更新网格内食物量
        my_cell.food <- my_cell.food - energy_transfert;
    } 
    //返回能量转移量         
    return energy_transfert;
    }
}

//创建食肉动物族，其父族为generic_species 
species predator parent: generic_species {
    //初始化显示大小
    float size <- 1.0;
    //初始化颜色
    rgb color <- #blue; 
    //初始化最大能量值
    float max_energy <- predator_max_energy ;
    //初始化每次最大进食量
    float max_transfert <- predator_max_transfert ;
    //初始化每次能量消耗值
    float energy_consum <- predator_energy_consum ;
    //初始化繁殖概率
    float proba_reproduce <- predator_proba_reproduce ;
    //初始化最大繁殖数
    int nb_max_offsprings <- predator_nb_max_offsprings ;
    //初始化繁殖能量
    float energy_reproduce <- predator_energy_reproduce ;
    //重写energy_from_eat函数
    float energy_from_eat {
    //初始化食肉动物能量转移量
    float energy_transfert <- predator_energy_transfert ;
    //列出所在网格内的食草动物
    list<prey> reachable_preys <- prey inside (my_cell); 
    //如果食草动物列表不为空   
    if(! empty(reachable_preys)) {
        //随机吃掉一个食草动物，那个食草动物死去
        ask one_of (reachable_preys) {
        do die;
        }
        //返回食肉动物每次进食量
        return energy_transfert;
    }
    //如果食草动物列表为空，返回0
    return 0.0;
    }
}

//创建一个50x50的四边形网格（注：neighbors控制网格的形状，如6边形、8边形等）
grid vegetation_cell width: 50 height: 50 neighbors: 4 {
	//定义网格属性
	//food代表每个网格的食物量
	//max_food每个网格最大食物量
	float max_food <- 1.0;
	//每次模拟网格中食物量随机增加的数值（0-0.01）
	float food_prod <- rnd(0.01);
	//每个网格初始食物量为（0-1）的随机数，每次模拟更新加food_prod，最大值为max_food
	float food <- rnd(1.0) max: max_food update: food + food_prod;
	//将周边距离自己为2的其他vegetation_cell存到neighbors2列表中
	list<vegetation_cell> neighbors2  <- (self neighbors_at 2);
	//根据食物量的大小，网格的颜色也会发生变化
	rgb color <- rgb(int(255 * (1 - food)), 255, int(255 * (1 - food))) update: rgb(int(255 * (1 - food)), 255, int(255 * (1 - food)));
}

//实验名称为prey_predator，输出方式为图形界面
experiment prey_predator type: gui {
	//定义食草动物可在图形界面调整的参数
	parameter "Initial number of preys: " var: nb_preys_init min: 1 max: 1000 category: "Prey";
	parameter "Prey max energy: " var: prey_max_energy category: "Prey";
	parameter "Prey max transfert: " var: prey_max_transfert category: "Prey";
	parameter "Prey energy consumption: " var: prey_energy_consum category: "Prey";
	parameter 'Prey probability reproduce: ' var: prey_proba_reproduce category: 'Prey';
	parameter 'Prey nb max offsprings: ' var: prey_nb_max_offsprings category: 'Prey';
	parameter 'Prey energy reproduce: ' var: prey_energy_reproduce category: 'Prey';
	//定义食肉动物可在图形界面调整的参数
	parameter "Initial number of predators: " var: nb_predators_init min: 0 max: 200 category: "Predator";
	parameter "Predator max energy: " var: predator_max_energy category: "Predator";
	parameter "Predator energy transfert: " var: predator_energy_transfert category: "Predator";
	parameter "Predator energy consumption: " var: predator_energy_consum category: "Predator";
	parameter 'Predator probability reproduce: ' var: predator_proba_reproduce category: 'Predator';
	parameter 'Predator nb max offsprings: ' var: predator_nb_max_offsprings category: 'Predator';
	parameter 'Predator energy reproduce: ' var: predator_energy_reproduce category: 'Predator';
	//定义输出
	output {
		//输出窗口名为main_display
		display main_display {
			//显示vegetation_cell网格，线型为黑色
			grid vegetation_cell lines: #black;
			//以prey族中aspect定义的base方式显示prey族
			species prey aspect: base;
		}
	}
}
```
