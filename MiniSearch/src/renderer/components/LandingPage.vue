<template>
  <div id="wrapper">
    <main>
	 <div id="selectClass">
		 <el-input id="selectInput" placeholder="请输入搜索目录" v-model="inputDirPath">
		  	<el-button round slot="append" size="small" @click=showFileDialog() type="primary">选择目录</el-button>			
			<el-select slot="prepend" width="100px" id="selectpath" v-model="selectTF" placeholder="常用目录" @change="changeValue($event)">
			   <el-option
			     v-for="item in options"
			     :key="item.value"
			     :label="item.label"
			     :value="item.value">
			   </el-option>
			 </el-select>
		  </el-input>
	 </div>
	 <br />
     <div>
		<el-input placeholder="请输入搜索内容(空字符搜索所有)" @keyup.enter.native="startSearch" v-model="searchTF">
		 <el-button slot="append" @click=startSearch() icon="el-icon-search"></el-button>
		</el-input>
	 </div>
	 <!-- <el-button type="primary" @click=startSetting() icon="el-icon-edit" circle></el-button> -->
		<template>
		  <el-table
			:data="tableData"
			stripe
			style="width: 100%"
			:max-height="400"
			highlight-current-row
			:row-style="{height:'44px'}"
			:cell-style="{padding:'0px'}"
			>
			<!-- 索引 -->
		   <el-table-column
			  type="index"
			  width="50px">
			</el-table-column>
			<!-- 使用方法 -->
			<el-table-column label="地址">
			  <template slot-scope="scope" prop="address">
			   <span v-html="brightenKeyword(scope.row.address,searchTF)" ></span>
			  </template>
			</el-table-column>
			
			 <el-table-column width="100px" align="center" label="操作">
				<template slot-scope="scope">
				<el-button size="mini" @click="handleShowInFinder(scope.$index, scope.row)" icon="el-icon-view" circle></el-button>
				</template>
			</el-table-column>
		  </el-table>
		</template>
    </main>
  </div>
</template>

<script>
  import SystemInformation from './LandingPage/SystemInformation'
	function run_spawnCmd(cmd,args,callback) {
		const { spawn } = require('child_process');
		const top = spawn(cmd,args);
		top.stdout.on('data', (data) => {
			callback(`${data}`)
		});
	
		top.stderr.on('data', (data) => {
			console.log(`stderr:\n ${data}`);
		});
	
		top.on('close', (code) => {
			callback(`${code}`)
			console.log(`child process exited with code ${code}`);
		});
	}
	
  export default {
    name: 'landing-page',
    components: { SystemInformation },
	data() {
		const os = require('os')
		const homeDir = os.homedir()
		const remote = require('electron').remote;
		const app = remote.app;
		const localStorage = require('electron-localstorage')
		console.log(app.getPath('desktop'));
		console.log(app.getPath('documents'));
		console.log(app.getPath('downloads'));
		var inputDirPath = localStorage.getItem("inputDirPath");//获取b的值,为"isaac"
		if (inputDirPath&&(inputDirPath.length > 1)) {
			console.log("filenameCacae:" + inputDirPath);
		}else{
			console.log("999999999")
			inputDirPath= `${homeDir}/Documents/精选图片库`
		}
		
		return {
			searchTF: "",
			inputDirPath:inputDirPath,
			tableData: [],
			options: [{
				value: 'desktop',
				label: app.getPath('desktop')
			}, {
				value: 'documents',
				label: app.getPath('documents')
			}, {
				value: 'downloads',
				label: app.getPath('downloads')
			}],
			selectTF: '',
			clientHeight: document.body.clientHeight,
		}
	},
	created() {
	    const documentHeight = document.body.clientHeight;
	    this.tableHeight = parseInt(documentHeight  *  0.8  -  100); // 计算表高度，固定表头
	},
	computed:{
		// console.log("======77777777=========");
	},
	
    methods: {
      open (link) {
        this.$electron.shell.openExternal(link)
      },
	  mounted() {
	      // 在DOM渲染数据时，设置下区域高度为浏览器可视区域高度．
	      this.clientHeight = document.body.clientHeight;
	      // 监听window的resize事件．在浏览器窗口变化时再设置下区域高度．
	      const _this = this;
	      window.onresize = function temp() {
			  console.log("888888onresize888888888888");
	          _this.clientHeight = 400;
			   // document.body.clientHeight-600;
	      };
	  },
	  startSearch(){
		var searchText=this.searchTF;
		let that = this
		var datalist=[]
		if(searchText.length<1){
			searchText=".";	//搜索所有
		}
		console.log(`开始搜索->${searchText}`);
		run_spawnCmd("/usr/local/bin/fd", [`${searchText}`,this.inputDirPath], function(data){
				if(data=="0"){
					console.log("00000000Datalist:"+datalist)
					that.tableData=datalist;
				}else{
					console.log("999999999:"+data)
					var files=data.split("\n"); //字符分割
					//过滤空元素
					files = files.filter(function(n){return n}); // (javascript 1.6 and above)  
					var tabData=[]
					files.forEach(function(item) {
						var dict={
							"address":`${item}`
						}
						tabData.push(dict)					
					})
					Array.prototype.push.apply(datalist, tabData);
				}
			}
		);
	  },
	  // 显示目录选择器
	  showFileDialog() {
		  const remote = require('electron').remote;
		  const app = remote.app;
		  console.log(app.getPath('userData'));
		  
	  	const dialog = require('electron').remote.dialog
	  	dialog.showOpenDialog({
	  		properties: ['openDirectory'],
	  		// filters: [{
	  		// 	name: 'IPAType',
	  		// 	extensions: ['ipa']
	  		// }]
	  	}, (filename) => {
	  		// this.$data.inputDirPath = filename[0]
	  		if (filename&&(filename.length == 1)) {
	  			this.inputDirPath = filename[0];
				console.log("filename:" + filename);
				localStorage.setItem("inputDirPath",this.inputDirPath);//设置b为"isaac"
				var inputDirPath00 = localStorage.getItem("inputDirPath");//获取b的值,为"isaac"
				console.log("inputDirPath00:" + inputDirPath00);
	  		}
	  	})
	  },
	  //Show in Finder
	  handleShowInFinder(index,row) {
		const shell = require('electron').shell
		const os = require('os')
		var dict=this.tableData[index]
		console.log(dict["address"])
		shell.showItemInFolder(dict["address"])
	  },
	  
	  // 获取select的label值和value值
	  changeValue(value) {
	      console.log(value);
	        let obj = {};
	        obj = this.options.find((item)=>{
	            return item.value === value;
	        });
	        console.log(obj.label);
			this.inputDirPath=obj.label
	  },
		// 筛选变色
		brightenKeyword(val, keyword) {
			// console.log("====================val+:"val);
			if(keyword.length<2){
				return val;
			}
			const Reg = new RegExp(keyword, 'ig');
			if (val) {
				const res = val.replace(Reg, `<span style="color: red;">${keyword}</span>`);
				return res;
			}
		},
		
		startSetting(){			
			const remote = require('electron').remote
			const { app, dialog, BrowserWindow } = remote
			// let top = remote.getCurrentWindow();
			// // 在这个例子中,`win` 是我们的实例
			// let win = new BrowserWindow({ 
			// 	parent: top,
			// 	width: 800, 
			// 	height: 600 ,
			// 	modal: true, // 设为模态窗口
			// 	})
			// win.on('close',()=>{
			// 	win=null
			// })
			 // Menu.setApplicationMenu(null) // 关闭子窗口菜单栏
			const winURL = process.env.NODE_ENV === 'development'
			  ? `http://localhost:9080`
			  : `file://${__dirname}/index.html`
			const modalPath = winURL+"#/AboutPage"
			// /Users/jenkins/DevelopFolder/Electron/AppUploader/src/renderer/views/About.vue
			console.log("modalPath:"+modalPath);
			let mainWindow = remote.getCurrentWindow();
			
			mainWindow.loadURL(modalPath)
			// return;
			
			// 使用hash对子页面跳转，这是vue的路由思想
			 let miniWindow = new BrowserWindow({
			    width: 600,
			    height: 400,
			    // webPreferences: {
			    //   webSecurity: false
			    // },
				 webPreferences: {
				      nodeIntegration: true
				    },
			    parent: mainWindow // mainWindow是主窗口
			  })

			  miniWindow.loadURL(modalPath)
			  miniWindow.on('closed', () => {
			    miniWindow = null
			  })
		}
		
		//================= 
    }
  }
</script>

<style>
  @import url('https://fonts.googleapis.com/css?family=Source+Sans+Pro');

  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  body { font-family: 'Source Sans Pro', sans-serif; }

  #wrapper {
    background:
      radial-gradient(
        ellipse at top left,
        rgba(255, 255, 255, 1) 40%,
        rgba(229, 229, 229, .9) 100%
      );
    height: 100vh;
    padding: 30px 30px;
    width: 100vw;
  }

  #logo {
    height: auto;
    margin-bottom: 20px;
    width: 420px;
  }
  

  

  main {
    /* display: flex; */
    justify-content: space-between;
  }

  main > div { flex-basis: 50%; }

  .left-side {
    display: flex;
    flex-direction: column;
  }
  
  /* el-input 前面是个 el-select，但是内容显示不出来*/
  .el-select .el-input {
	width: 105px;
  }
  
/*  #selectClass {
  	  display: flex;
  } */
  
/*  #selectInput {
	  width: 80%;
  }
 */
  .welcome {
    color: #555;
    font-size: 23px;
    margin-bottom: 10px;
  }

  .title {
    color: #2c3e50;
    font-size: 20px;
    font-weight: bold;
    margin-bottom: 6px;
  }

  .title.alt {
    font-size: 18px;
    margin-bottom: 10px;
  }

  .doc p {
    color: black;
    margin-bottom: 10px;
  }

  .doc button {
    font-size: .8em;
    cursor: pointer;
    outline: none;
    padding: 0.75em 2em;
    border-radius: 2em;
    display: inline-block;
    color: #fff;
    background-color: #4fc08d;
    transition: all 0.15s ease;
    box-sizing: border-box;
    border: 1px solid #4fc08d;
  }

  .doc button.alt {
    color: #42b983;
    background-color: transparent;
  }
</style>
