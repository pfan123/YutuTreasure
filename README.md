YutuTreasure
============

玉兔寻宝，带你一起遨游太空，历经艰险，寻找宝藏，好玩的游戏，你敢来挑战！                                           

访问地址：http://pingfan1990.sinaapp.com/html5/pfgame/index.html

玉兔寻宝，是一款微信小游戏，采用html5+canvas制作，没有使用框架，性能还需要优化。

主要部分代码带注释展示：

//效果最好的动画方法兼容写法	
		window.RAF = (function(){
			return window.requestAnimationFrame || window.webkitRequestAnimationFrame || window.mozRequestAnimationFrame || window.oRequestAnimationFrame || window.msRequestAnimationFrame || function (callback) {window.setTimeout(callback, 1000 / 60); };
        })();	
			
		//Ship飞船
		function Ship(cxt){
			this.width=60,
			this.height=60,
			this.top=stage.height-this.height-16,
			this.left=stage.width/2-this.width/2;
			this.loading=false,
			this.loadimages={};
			var player=new Image();
			this.paint = function(){
				var _this=this;
				this.player=player;
				player.addEventListener('load',function(){
					cxt.drawImage(player, _this.left, _this.top, _this.width, _this.height);
				},false);
				player.src='img/player.png';
				
			},
			//换用chongpaint相当于做图片延迟加载器
			this.chongpaint=function(){
				cxt.drawImage(this.player, this.left, this.top, this.width, this.height);
			},
			//设置位置
			this.setPosition=function(event){
				if(pingfan.isMobile()){
					var tarL = event.changedTouches[0].clientX;
					var tarT = event.changedTouches[0].clientY;
				}else{
					var tarL = event.offsetX;
					var tarT = event.offsetY;
				}
				this.left = tarL - this.width/2 - 16;
				this.top = tarT - this.height/2;
				if(this.left<0){
					this.left = 0;
				}
				if(this.left>stage.width-this.width){
					this.left = stage.width-this.width;
				}
				if(this.top<0){
					this.top = 0;
				}
				if(this.top>stage.height - this.height){
					this.top = stage.height - this.height;
				}
				//cxt.clearRect(0,0,stage.width,stage.height);
				this.chongpaint();			
			},
			
			//事件控制
			this.controll=function(){
				var currentX = this.left,
					currentY = this.top,
			        _this=this,move = false;;
				stage.addEventListener(touchStart,function(event){
			    	//event.preventDefault();
					_this.setPosition(event);
					move=true;
				},false);	
				stage.addEventListener(touchEnd,function(){
					move=false;
				},false);	
				stage.addEventListener(touchMove,function(event){
					event.preventDefault();
					if(move){
					 _this.setPosition(event);
					}
				},false);					
			},
			
			//吃的方法
			this.eat=function(fallList){
				for(var i=fallList.length;i--;){
					var f = fallList[i];
					if(f){
						var a=this.top+this.height/2 - (f.top+f.height/2),
							b=this.left+this.width/2- (f.left+f.width/2),
							average=Math.sqrt(a*a+b*b);
						//出现碰撞	
						if(average<=(this.height/2+f.height/2)){
						
							fallList[f.id] = null;
							if(f.type==0){
								console.log(1);
								gameMonitor.stop();
								fallList[f.id] = null;
							}else{
								score.innerHTML=++gameMonitor.score;
								pingfan.addClass(heart,'hotheart');
								setTimeout(function() {
									pingfan.removeClass(heart,'hotheart');
								}, 200);
							}
						}	
					}
				}
			}
		}		
		
		//下落
		function Fallingstone(type, left, id){
			this.speedUpTime = 2000;
			this.id = id;
			this.type = type;
			this.width = 50;
			this.height = 50;
			this.left = left;	
			this.top = -50;	
			this.speed = 0.01 * Math.pow(1.1, Math.floor(gameMonitor.time/this.speedUpTime));
			this.loop = 0;
			var p = this.type == 0 ? 'img/food1.png' : 'img/food2.png',			
			    pic=new Image();
				pic.src=p;
			this.pic=pic;
		}
		
		Fallingstone.prototype={
			constructor:Fallingstone,
			paint:function(cxt){
				cxt.drawImage(this.pic, this.left, this.top, this.width, this.height);
			},
			move:function(cxt){
				if(gameMonitor.time % this.speedUpTime == 0){
					this.speed *= 0.8;
				}
				this.top += ++this.loop * this.speed;
				if(this.top>gameMonitor.bgheight){
					gameMonitor.fallList[this.id] = null;
				}
				else{
					this.paint(cxt);
				}			
			}
		}
				
		//gameMonitor游戏场景
		var gameMonitor={
			bgwidth:stage.width,
			bgheight:stage.height,
			time : 0,
			timmer : null,	
			bgSpeed : 2,//移动增长值
			bgloop : 0,	
			bgDistance : 0,//背景位置,
			fallList : [],
			score:0,
			initStart:true,
			init:function(){
				var cxt=stage.getContext('2d');
				var bg=new Image(),_this=this;
				this.bg = bg;
				
				//图片加载成功，绘画背景
				bg.addEventListener('load',function(){
					cxt.drawImage(bg,0,0,stage.width,stage.height);
				},false);
				bg.src='img/bg.jpg';
				this.initListener(cxt);
			},
			initListener:function(cxt){
				var _this=this;
				
				//隐藏指示，开始
				guidecontainer.addEventListener(touchStart,function(){
					audio.play();
					guidecontainer.style.display='none';
					_this.ship=new Ship(cxt);
					_this.ship.paint();
					_this.ship.controll();
					_this.reset();
					_this.run(cxt);
				},false);
				
				//再来一次
				replay.addEventListener(touchStart,function(){
					audio.play();
					gameresult.style.display='none';
					_this.ship=new Ship(cxt);
					_this.ship.paint();
					_this.ship.controll();
					_this.reset();
					_this.run(cxt);	
				},false);
				
				sharebutton.addEventListener(touchStart,function(){
					if(pingfan.isWeiXin())share.style.display='block';
					//share.style.display='block';
				},false)
				share.addEventListener(touchStart,function(){
					share.style.display='none';
				},false);
				
			},
			//动态背景
			rollBg : function(cxt){
				if(this.bgDistance>=this.bgheight){
					this.bgloop = 0;
				}
				this.bgDistance = ++this.bgloop * this.bgSpeed;
				cxt.drawImage(this.bg, 0, this.bgDistance-this.bgheight, this.bgwidth, this.bgheight);
				cxt.drawImage(this.bg, 0, this.bgDistance, this.bgwidth, this.bgheight);
			},
			run:function(cxt){
				var _this=this;
				cxt.clearRect(0,0,_this.bgwidth,_this.bgheight);		
				_this.rollBg(cxt);
										
				//绘制飞船
				_this.ship.chongpaint();
				
				_this.ship.eat(_this.fallList);
				//console.log(_this.fallList);
				//产生月饼
				_this.genorateFall();

				//绘制月饼
				for(i=_this.fallList.length-1; i>=0; i--){
					var f = _this.fallList[i];
					if(f){
						f.paint(cxt);
						f.move(cxt);
					}
					
				}				
				
				if(_this.initStart)_this.timmer = RAF(function(){_this.run(cxt);});
				_this.time++;					
			},
			reset:function(){
				this.fallList = [];
				this.bgloop = 0;
				this.score = 0;
				this.timmer = null;
				this.time = 0;	
				this.initStart=true;
				score.innerHTML=0;				
			},
			//生产陨石
			genorateFall : function(){
				var genRate = 60; //产生月饼的频率
				var random = Math.random();
				if(random*genRate>genRate-1){
					var left = Math.random()*(this.bgwidth - 60);
					var type = Math.floor(left)%2 == 0 ? 0 : 1;
					var id = this.fallList.length;
					var f = new Fallingstone(type, left, id);
					this.fallList.push(f);
				}
			},
			stop:function(){
				audio.pause();
				var _this = this;
				console.log(111);
				gameover.style.display='block';
				_this.initStart=null;
				setTimeout(function(){
					gameover.style.display='none',
					_this.getScore(),
					gameresult.style.display='block';
				},1000);
				
			},
			getScore:function(){
				var time=Math.floor(this.time/60);
				if(this.score==0){
					scorecontent.innerHTML="真遗憾，你既然<span class='lighttext'>一个</span>也没有捞到。";
					suclog.className='';
					suclog.className='yihan';			
					shareData.tContent="你也太菜了吧，你既然一个也没有捞到，你也来试试吧。"
					return;
				}else if(this.score<10){
					scorecontent.innerHTML="您在<span class='lighttext'>"+time+"</span>秒内捞到了<span class='lighttext'>"+this.score+"</span>个宝贝，超过了<span class='lighttext'>2%</span>的用户！";
					shareData.tContent="我在"+time+"秒内捞到了"+this.score+"个宝贝，超过了2%的用户，敢跟我来比一比吗？"
				}else if(this.score>=10 && this.score<20){
					scorecontent.innerHTML="您在<span class='lighttext'>"+time+"</span>秒内捞到了<span class='lighttext'>"+this.score+"</span>个宝贝，超过了<span class='lighttext'>10%</span>的用户！";
					shareData.tContent="我在"+time+"秒内捞到了"+this.score+"个宝贝，超过了10%的用户，敢跟我来比一比吗？"
				}else if(this.score>=20 && this.score<40){
					scorecontent.innerHTML="您在<span class='lighttext'>"+time+"</span>秒内捞到了<span class='lighttext'>"+this.score+"</span>个宝贝，超过了<span class='lighttext'>40%</span>的用户！";
					shareData.tContent="我在"+time+"秒内捞到了"+this.score+"个宝贝，超过了40%的用户，敢跟我来比一比吗？"
				}else if(this.score>=40 && this.score<60){
					scorecontent.innerHTML="您在<span class='lighttext'>"+time+"</span>秒内捞到了<span class='lighttext'>"+this.score+"</span>个宝贝，超过了<span class='lighttext'>60%</span>的用户！";
					shareData.tContent="我在"+time+"秒内捞到了"+this.score+"个宝贝，超过了60%的用户，敢跟我来比一比吗？"
				}else if(this.score>=60 && this.score<90){
					scorecontent.innerHTML="太棒了，您在<span class='lighttext'>"+time+"</span>秒内捞到了<span class='lighttext'>"+this.score+"</span>个宝贝，超过了<span class='lighttext'>80%</span>的用户！";
					shareData.tContent="太棒了，我在"+time+"秒内捞到了"+this.score+"个宝贝，超过了80%的用户，敢跟我来比一比吗？"
				}else if(this.score>=90 && this.score<=120){
					scorecontent.innerHTML="太棒了，您在<span class='lighttext'>"+time+"</span>秒内捞到了<span class='lighttext'>"+this.score+"</span>个宝贝，超过了<span class='lighttext'>90%</span>的用户！";
					shareData.tContent="太棒了，我在"+time+"秒内捞到了"+this.score+"个宝贝，超过了90%的用户，敢跟我来比一比吗？"
				}else if(this.score>150){
					scorecontent.innerHTML="太棒了，您在<span class='lighttext'>"+time+"</span>秒内捞到了<span class='lighttext'>"+this.score+"</span>个宝贝，超过了<span class='lighttext'>99%</span>的用户！";
					shareData.tContent="太棒了，我在"+time+"秒内捞到了"+this.score+"个，超过了99%的用户，敢跟我来比一比吗？"
				}
				suclog.className='';
				suclog.className='good';	
			}
		}
		
		
		
		
	主要封装了几个对象，服从主对象gameMonitor游戏场景。

  亲，喜欢就点星、点赞，也希望各为大神指点、交流。	
