### 从一个菊花开始说起


由于我的硬盘空间比较小，然后资源比较多。所以一直在用一个管理硬盘资源的软件 叫做 Daisy（菊花）,UI 大概是下面这个样子。 （我再演示一下具体的效果）
![](//47.98.47.153/1533690455062.png)

恩。。。整体上就是感觉很酷，所以我就准备看看能不能自己做一个。

后面在可视化里面找到了这个东西的学名，它不叫菊花，应该称之为 旭日图（虽然字面意思也感觉差不多）

旭日图  是一种比较现代化的饼图，可以用饼图的方式展示多层级数据，用于比较。

![](//47.98.47.153/1533691083517.png)


### 从一个canvan面板到一个扇形

恩。。。刚开始应该怎么做呢，其实也没一个很好的思路，但是这个东西应该是从一个饼图演化来的，之前我也没有纯手工画图表的经验。那么我先DIY一个饼图（pie）出来吧。 需要画出一个饼图，那么先画一个扇形吧（为什么不从圆开始？）

上1-1的代码


```
 <canvas id="myCanvas" width="900" height="900" style="background: #f3f3f3;">
        您的浏览器不支持canvas标签。
    </canvas>
    <script>
        var myCanvas = document.getElementById('myCanvas');
        var ctx = myCanvas.getContext('2d');

        function pie() {
            this.r = 50;
        }

        // 扇形
        pie.prototype.draw = function (startAngle, endAngle) {
            ctx.beginPath();
            ctx.arc(100, 75, this.r, startAngle || 0, endAngle || 2 * Math.PI);
            ctx.fillStyle = "red";
            ctx.fill();
            ctx.closePath();
        }
        new pie().draw(Math.PI / 3, 2 * Math.PI / 3);
    </script>

```

用到的canvas API --- ARC

![](//47.98.47.153/1533691552518.png)


我以为的效果  
![](//47.98.47.153/1533691978916.png)

------
 实际的效果
![](//47.98.47.153/1533692023408.png)

![](//47.98.47.153/1533692263661.jpg)


所以这个扇形比想象的要麻烦，ARC画出来的是弧形，这样是正确的，我的理解存在误区，继续上1-2的代码

思路应该是 三角形+弧形(突然代码多了很多)

```
 <canvas id="myCanvas" width="900" height="900" style="background: #f3f3f3;">
        您的浏览器不支持canvas标签。
    </canvas>

    <script>
        /*
         * 绘制点,并且在画板上显示点的名字和坐标
         *
         * point
         *  x,y,name
         *
         *  如果name为空 则不会显示坐标和名字
         * */

        function fillPoint(ctx, point) {
            if (!ctx || !point) {
                return false;
            }
            ctx.beginPath();
            ctx.arc(point.x, point.y, 2, 0, 2 * Math.PI, true);
            ctx.fill();
            ctx.font = "36px";
            if (point.name) {
                ctx.fillText(point.name + '(' + point.x + ',' + point.y + ')', point.x + 10, point.y);
            }
            ctx.closePath();
        }

        /*
         *  传入数组,进行连线
         * */

        function linePoint(ctx, PointList) {
            if (!ctx || !PointList.length) {
                return false;
            }
            ctx.beginPath();
            for (var i = 0; i < PointList.length; i++) {
                var point = PointList[i];

                if (i == 0) {
                    ctx.moveTo(point.x, point.y);
                } else {
                    ctx.lineTo(point.x, point.y);
                }
            }
            ctx.stroke();
            ctx.closePath();
        }
    </script>
    <script>
        var myCanvas = document.getElementById('myCanvas');
        var ctx = myCanvas.getContext('2d');

        function pie() {
            this.r = 50; //半径
            this.r_point = {
                x: 100,
                y: 75,
                name: '圆心'
            }
        }

        // http://www.w3school.com.cn/tags/canvas_arc.asp
        // 先画个框架
        pie.prototype.drawSB = function (startAngle, endAngle) {
            const self = this;
            ctx.beginPath();
            // 绘制关键点
            fillPoint(ctx, self.r_point);

            const start_point = {
                x: self.r_point.x + self.r * Math.cos(startAngle),
                y: self.r_point.y + self.r * Math.sin(startAngle),
                name: '起点'
            }
            fillPoint(ctx, start_point);

            const end_point = {
                x: self.r_point.x + self.r * Math.cos(endAngle),
                y: self.r_point.y + self.r * Math.sin(endAngle),
                name: '终点'
            }
            fillPoint(ctx, end_point);

            linePoint(ctx, [self.r_point, start_point, end_point]);

            ctx.arc(100, 75, this.r, startAngle || 0, endAngle || 2 * Math.PI);
            ctx.fillStyle = "red";
            ctx.stroke();


            ctx.fill();




            ctx.closePath();
        }
        new pie().drawSB(Math.PI / 3, 2 * Math.PI / 3);
    </script>


```

辅助的图：
1> canvas 的坐标系![](//47.98.47.153/1533692789634.png)
2> 求 start_point  和  end_point 的辅助图![](//47.98.47.153/1533693417029.jpg)



###  从一个扇形到一个饼图

这个过程就比较简单了，只需要传入一个数组的数据，然后把一个圆分成N份就可以了

上2-1的代码

```
 <script>
        var myCanvas = document.getElementById('myCanvas');
        var ctx = myCanvas.getContext('2d');

        function pie() {
            this.r = 50; //半径
            this.r_point = {
                x: 100,
                y: 75,
                name: '圆心'
            }
            this.data = [10, 20, 30];
        }

        // http://www.w3school.com.cn/tags/canvas_arc.asp
        // 先画个框架
        pie.prototype.drawSB = function (
            startAngle,
            endAngle,
            color, debug) {
            const self = this;
            // 绘制关键点
            ctx.beginPath();
            const start_point = {
                x: self.r_point.x + self.r * Math.cos(startAngle),
                y: self.r_point.y + self.r * Math.sin(startAngle),
                name: '起点'
            }
            const end_point = {
                x: self.r_point.x + self.r * Math.cos(endAngle),
                y: self.r_point.y + self.r * Math.sin(endAngle),
                name: '终点'
            }

            linePoint(ctx, [self.r_point, start_point, end_point]);


            ctx.arc(100, 75, this.r, startAngle || 0, endAngle || 2 * Math.PI);
            ctx.fillStyle = color;
            ctx.fill();
            ctx.closePath();

            if (debug) {
                fillPoint(ctx, start_point);
                fillPoint(ctx, end_point);
                console.log('start_point', start_point)
                console.log('end_point', end_point)
            }

        }
        pie.prototype.fillData = function () {
            const self = this;
            const data = this.data;
            const sum = this.data.reduce((prev, cur) => {
                return prev + cur;
            });
            const colorMap = ['red', 'green', 'gray'];

            let startAngle = 0;
            data.map((item, index) => {
                const percent = item / sum;
                const endAngle = startAngle + 2 * percent * Math.PI;
                console.log(endAngle);
                self.drawSB(startAngle, endAngle, colorMap[index]);
                startAngle = endAngle;
            })
        }

        new pie().fillData();
    </script>
```


然后整理一下参数，把变量放到pie的option里面，并且加一些备注

```
 <script>
        var myCanvas = document.getElementById('myCanvas');
        var ctx = myCanvas.getContext('2d');

        /*  Pie 饼图Class
         *  options.r  半径
         *  options.r_point  圆点  { x:20,y:20,name:'圆心'}
         *  options.colorMap  颜色数组   ['green', 'red', 'blue']
         *  options.data  数据   [20,30,60]
         */
        function Pie(options) {
            this.r = options.r;
            this.r_point = options.r_point;
            this.colorMap = options.colorMap;
            this.data = options.data;
            this.fillData();
        }

        // http://www.w3school.com.cn/tags/canvas_arc.asp
        // 绘制扇形
        Pie.prototype.drawSB = function (
            startAngle,
            endAngle,
            color, debug) {
            const self = this;
            // 绘制关键点
            ctx.beginPath();
            const start_point = {
                x: self.r_point.x + self.r * Math.cos(startAngle),
                y: self.r_point.y + self.r * Math.sin(startAngle),
                name: '起点'
            }
            const end_point = {
                x: self.r_point.x + self.r * Math.cos(endAngle),
                y: self.r_point.y + self.r * Math.sin(endAngle),
                name: '终点'
            }

            linePoint(ctx, [self.r_point, start_point, end_point]);


            ctx.arc(self.r_point.x, self.r_point.y, this.r, startAngle || 0, endAngle || 2 * Math.PI);
            ctx.fillStyle = color;
            ctx.fill();
            ctx.closePath();

            if (debug) {
                fillPoint(ctx, start_point);
                fillPoint(ctx, end_point);
                console.log('start_point', start_point)
                console.log('end_point', end_point)
            }

        }

        // 填入data
        Pie.prototype.fillData = function () {
            const self = this;
            const data = this.data;
            const sum = this.data.reduce((prev, cur) => {
                return prev + cur;
            });

            let startAngle = 0;
            data.map((item, index) => {
                const percent = item / sum;
                const endAngle = startAngle + 2 * percent * Math.PI;
                console.log(endAngle);
                self.drawSB(startAngle, endAngle, self.colorMap[index]);
                startAngle = endAngle;
            })
        }
    </script>

    <script>
        const pie1 = new Pie({
            r: 50,
            r_point: {
                x: 200,
                y: 200,
                name: '圆心'
            },
            colorMap: ['green', 'red', 'blue'],
            data: [20, 30, 60]
        })
    </script>
```

###  从一个饼图到一个菊花图


现在我就发现了，其实旭日图就是 很多个不同R的饼图的叠加。 R的距离 暂定是等差的。 
2-2 就把3个Pie 放到了一起，其他不变 看看效果

```
 <script>
        const pie1 = new Pie({
            r: 50,
            r_point: {
                x: 200,
                y: 200,
                name: '圆心'
            },
            colorMap: ['green', 'red', 'blue'],
            data: [20, 30, 60]
        })

        const pie2 = new Pie({
            r: 25,
            r_point: {
                x: 200,
                y: 200,
                name: '圆心'
            },
            colorMap: ['grey', 'green', 'aqua'],
            data: [10, 10, 10]
        })

        const pie3 = new Pie({
            r: 10,
            r_point: {
                x: 200,
                y: 200,
                name: '圆心'
            },
            colorMap: ['beige'],
            data: [1]
        })
    </script>
```

效果图
![](//47.98.47.153/1533694275549.png)


最后2-4 旭日图对象就出来了，data 是一个树状结构，从树里面拿到每个圈的占比，new 一个Pie

这里比较麻烦的是数据处理，需要把一个树状结构压平一下。

```
<script>
        var item1 = {
            color: '#F54F4A'
        };
        var item2 = {
            color: '#FF8C75'
        };
        var item3 = {
            color: '#FFB499'
        };

        var data = [{
            children: [{
                value: 5,
                children: [{
                    value: 1,
                    itemStyle: item1
                }, {
                    value: 2,
                    children: [{
                        value: 1,
                        itemStyle: item2
                    }]
                }, {
                    children: [{
                        value: 1
                    }]
                }],
                itemStyle: item1
            }, {
                value: 10,
                children: [{
                    value: 6,
                    children: [{
                        value: 1,
                        itemStyle: item1
                    }, {
                        value: 1
                    }, {
                        value: 1,
                        itemStyle: item2
                    }, {
                        value: 1
                    }],
                    itemStyle: item3
                }, {
                    value: 2,
                    children: [{
                        value: 1
                    }],
                    itemStyle: item3
                }, {
                    children: [{
                        value: 1,
                        itemStyle: item2
                    }]
                }],
                itemStyle: item1
            }],
            value: 15,
            itemStyle: item1
        }, {
            value: 9,
            children: [{
                value: 4,
                children: [{
                    value: 2,
                    itemStyle: item2
                }, {
                    children: [{
                        value: 1,
                        itemStyle: item1
                    }]
                }],
                itemStyle: item1
            }, {
                children: [{
                    value: 3,
                    children: [{
                        value: 1
                    }, {
                        value: 1,
                        itemStyle: item2
                    }]
                }],
                itemStyle: item3
            }],
            itemStyle: item2
        }, {
            value: 7,
            children: [{
                children: [{
                    value: 1,
                    itemStyle: item3
                }, {
                    value: 3,
                    children: [{
                        value: 1,
                        itemStyle: item2
                    }, {
                        value: 1
                    }],
                    itemStyle: item2
                }, {
                    value: 2,
                    children: [{
                        value: 1
                    }, {
                        value: 1,
                        itemStyle: item1
                    }],
                    itemStyle: item1
                }],
                itemStyle: item3
            }],
            itemStyle: item2
        }, {
            children: [{
                value: 6,
                children: [{
                    value: 1,
                    itemStyle: item2
                }, {
                    value: 2,
                    children: [{
                        value: 2,
                        itemStyle: item2
                    }],
                    itemStyle: item1
                }, {
                    value: 1,
                    itemStyle: item3
                }],
                itemStyle: item3
            }, {
                value: 3,
                children: [{
                    value: 1,
                }, {
                    children: [{
                        value: 1,
                        itemStyle: item2
                    }]
                }, {
                    value: 1
                }],
                itemStyle: item3
            }],
            itemStyle: item3,
            value: 3
        }];


        //  旭日图 对象
        function sunburst() {
            this.data = data
            this.hanldeData();
        }

        sunburst.prototype.buildPie = function (options) {
            return new Pie({
                r: options.r,
                r_point: {
                    x: 200,
                    y: 200,
                    name: '圆心'
                },
                colorMap: options.colorMap,
                data: options.data
            })
        }
        sunburst.prototype.hanldeData = function () {
            const self = this;
            let data = self.data;
            let leave = 0;
            let obj = [{
                pieData: [1],
                colorMap: ['#f3f3f3'],
                leave: 0
            }]

            flattenData(data, leave + 1);

            // 将树状数据结构 压平成 分级数据，每一级的数据合并在一起
            function flattenData(data, leave) {
                const pieData = [];
                const colorMap = [];

                data.map(item => {
                    pieData.push(item.value || 0);
                    colorMap.push(item.itemStyle ? item.itemStyle.color : '#f3f3f3');
                    if (item.children) {
                        flattenData(item.children, leave + 1);
                    }
                })
                // 将当前数据填入对应的leave
                let cureentIndex;
                let currentLeaveData = obj.filter((item, index) => {
                    if (item.leave == leave) {
                        cureentIndex = index;
                        return true;
                    }
                });
                if (currentLeaveData.length > 0) {
                    obj[cureentIndex] = {
                        pieData: currentLeaveData[0].pieData.concat(pieData),
                        colorMap: currentLeaveData[0].colorMap.concat(colorMap),
                        leave
                    };
                } else {
                    obj.push({
                        leave,
                        colorMap,
                        pieData
                    });
                }
            }

            console.log(obj);

            for (let currentLeave = obj.length - 1; currentLeave >= 0; currentLeave--) {
                obj.map(item => {
                    if (item.leave == currentLeave) {
                        self.buildPie({
                            r: (item.leave + 1) * 30,
                            data: item.pieData,
                            colorMap: item.colorMap,
                        })
                    }
                })
            }
        }
        new sunburst();
    </script>
		
```


数据处理辅助图：

![](//47.98.47.153/1533695045379.jpg)

效果图

![](//47.98.47.153/1533695144966.png)


### 关于颜色的那些事情

### 处理事件

### 总结  

这个东西 可能刚开始不知道怎么去做，但是一步步分解 就变得 比较简单了。组件化的思路很重要，要把复杂的东西 拆分成比较简单的。