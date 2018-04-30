# 计算机系统结构 实验报告

计54 马子轩

## 实验目的

深入理解各种不同的 Cache 替换策略

理解学习不同替换策略对程序运行性能的影响

动手实现自己的 Cache 替换策略

## 策略分析

### LRU

Least Recently Used

每次miss的时候替换最不经常被使用的。实现上就是，实现一个队列，每次hit的时候把该项移动到最前面，miss的时候替换最后面的项目。

### Random

顾名思义，就是miss的时候随机选取一个进行替换

### Clock

LRU中，hit时需要进行扫描，为了减少开销，构造循环链表，命中的时候，置1。miss的时候顺序扫描，如果是1，置0，如果是0，替换。这种方式就能够减少扫描的开销。

### LIRS

根据hit间隔，来判断该cache块的访问频率，优先替换访问频率不高的块。具体实现就是，存放两个队列，一个用来存放低频块，一个用来存放所有块。hit的时候，如果在低频队列中，从低频队列中取出，加入总队列，同时从总队列中pop一个加入低频队列。如果不在低频队列中，直接取出，push进总队列。miss的时候，pop低频队列进行替换。

### 策略比较

LIRS相比LRU更多的考虑了访问频率的问题，在某些情况下性能更优。

考虑到cache替换策略是需要硬件实现的，而硬件中进行扫描比较，虽然速度较快，但是成本极高。所以LRU和LIRS这两种算法虽然性能比较好，但是在路比较多的时候成本较高。而clock性能虽然有所下降，但是成本较低。

## 策略设计

我的策略综合了clock和LIRS，使用两个clock来进行操作，命名为CLIRS。

hotclock存放所有cache块，coldclock存放所有冷cache块。

定义pop操作:按顺序扫描指针，如果1置0，如果0取出并停止。

hit: 如果在coldclock中存在，那么从hotclock中pop一个，替换该块在coldclock中位置，并置1，指针指向下一个。如果在coldclock中不存在，直接置1。

miss: 在coldclock中pop一个。替换

我认为我的设计，兼顾了LIRS的性能与Clock的效率。性能低于LISRS但是效率要高于LRU。

## 策略实现

Hit

```cpp
void CACHE_REPLACEMENT_STATE::UpdateCLIRS(UNIT32 setIndex, INT32 updateWayID) {
	int &hotpos = repl[setIndex][updateWayID].hotpos;
	int &coldpos = repl[setIndex][updateWayID].coldpos;

	hotclock[setIndex][hotpos].ref = 1;
	if (coldpos == -1) {
		int &pos = hothand[setIndex];
		while (hotclock[setIndex][pos].ref == 1) {
			hotclock[setIndex][pos].ref = 0;
		}
		int idx = hotclock[setIndex][pos].idx;
		int &pos = coldhand[setIndex];
		while (coldclock[setIndex][pos].ref == 1) {
			coldclock[setIndex][pos].ref = 0;
		}
		repl[setIndex][coldclock[setIndex][pos].idx].coldpos = -1;
		coldclock[setIndex][pos].idx = updateWayID;
	    repl[setIndex][updateWayID].coldpos = coldpos;
	}
}
```

Miss

```cpp
INT32 CACHE_REPLACEMENT_STATE::GetVictimCLIRE(UNIT32 setIndex) {
	int &pos = coldhand[setIndex];
	while (coldclock[setIndex][pos].ref == 1) {
		coldclock[setIndex][pos].ref = 0;
	}
	return coldclock[setIndex][pos].idx;
}
```

## 性能测试

|ID|400|401|403|410|416|429|433|434|435|436|437|444|445|447|450|453|454|456|458|459|462|464|465|470|471|473|482|483|999|
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
|LRU|40.7952|61.7369|42.7521|99.6634|26.3204|75.676|77.0164|80.81|71.1979|76.1164|84.0284|66.2603|14.919|45.0892|15.2011|39.9258|34.05|71.4736|93.5415|84.1525|100|70.0086|78.8694|99.9725|59.4443|5.84718|97.156|66.2759|95.9373|
|Random|44.4057|61.6783|45.6397|99.6634|27.6546|76.6152|76.2749|82.6395|74.3719|73.7824|80.9639|66.3123|20.4838|41.7856|17.5536|40.2017|38.3193|71.4736|94.1036|84.1525|100|71.7721|79.4314|99.9385|60.0557|5.84718|97.262|67.5795|96.3212|
|CLIRS|40.7506|61.5322|40.4667|99.6634|27.1623|68.0301|77.0904|80.6742|71.2816|69.3649|83.1827|66.2603|14.1997|41.5373|15.3786|39.8761|35.0522|71.4736|93.3517|84.1525|100|70.0602|78.8456|99.9691|59.4224|5.84718|97.1736|65.7373|95.9053|

在共29个测试数据中，CLIRS在22个测试中优于LRU，在26个测试点中优于Random，在20个测试点中同时优于LRU和Random。因此CLIRS在性能上稍优于LRU。

但是考虑到使用clock带来的成本降低，CLIRS是个较为不错的算法。

考虑到cache替换是由硬件进行实现的，模拟器中串行的扫描在硬件上实际是并行的，因此程序运行时间只能够一定程度上反应硬件的成本，没有实际测试意义，因此我没有进行计算。

## 实验总结

我在LIRS和clock的基础上设计了CLIRS算法，并进行了测试，在这个测试中我对cache的替换策略和cache的运行方式有了更深入的理解。同时，该实验有一定的局限性。正常的cache一般为4路，这时候进行LRU或LIRS的扫描成本并没有那么高，而16路组相联中，这带来的性能影响就比较大了。3个测试跑完要花一整天时间。可能将来设计时候可以更多的考虑实际cache的情况。