<template>
	<div class="layout">
		<Layout>
			<!-- 顶部导航栏 -->
			<Header>
				<Menu mode="horizontal" theme="dark">
					<div class="nav-head">
						<div class="nav-head-logo">IPCIS</div>
						<div class="nav-head-title">
							<div>国际化域名累计观测系统</div>
							<div>Accumulative International Domain Name Observation System</div>
						</div>
					</div>
				</Menu>
			</Header>

			<Layout>
				<!-- 侧边栏 -->
				<Sider hide-trigger style="background:#fff;">
					<Menu active-name="0" theme="light" width="auto" @on-select="route">
						<MenuItem name="3">
							<Icon type="ios-list"></Icon>
							已知域名列表
						</MenuItem>
						<MenuItem name="0">
							<Icon type="ios-information-circle-outline"/>
							域名基本信息
						</MenuItem>
						<MenuItem name="2">
							<Icon type="ios-locate"></Icon>
							通信对象地理分布
						</MenuItem>
						<MenuItem name="1">
							<Icon type="ios-git-network"></Icon>
							通信对象网络拓扑
						</MenuItem>
					</Menu>
				</Sider>

				<!-- 页面主体部分 -->
				<Content class="content">
					<Layout>
						<Breadcrumb class="current-target">
							<BreadcrumbItem>{{targetDomain}}</BreadcrumbItem>
							<BreadcrumbItem>{{pages[curModule][0]}}</BreadcrumbItem>
						</Breadcrumb>

						<Content class="content">
							<!-- 此处是各个模块组件的容器，实现切换 -->
							<keep-alive>
								<router-view/>
							</keep-alive>
						</Content>
					</Layout>
				</Content>
			</Layout>

			<!-- 底部footer -->
			<Footer class="footer">
				<div>
					2018 &copy; Silence-Rain<span>联系方式：daniel.s.mo503@gmail.com</span>
				</div>
			</Footer>
		</Layout>
	</div>
</div>
</template>

<script>
	export default {
		data () {
			return {
				targetDomain: "",			// 要查询的目标域名
				ips: [],					// 目标域名解析IP列表
				curModule: 0,				// 当前选中的模块
				pages: [					// 组件和模块名映射
					["域名基本信息", "BasicInfo"], 
					["通信对象网络拓扑", "TransTopo"],
					["通信对象地理位置分布", "GeoDistribution"]
				]
			}
		},

		mounted () {
			// 从localStorage中获取目标域名
			// 若localStorage中不存在，则根据上个页面路由中传递的参数设置目标域名
			this.targetDomain = localStorage.getItem("domain")
			if (this.targetDomain == null) {
				this.targetDomain = this.$route.params.domain_name
				localStorage.setItem("domain", this.targetDomain)
			}
			// 默认路由到“域名基本信息”模块
			this.route(0)
			// 监听子组件“域名基本信息”模块发送的域名解析IP事件，并更新自身数据和localStorage
			this.bus.$on("resolved_ips", (ips) => {
				localStorage.setItem("ips", JSON.stringify(ips))
			})
		},

		methods: {
			// 切换模块的路由函数
			route (name) {
				// 切换到“已知域名列表”模块之前，清空localStorage
				if (name == 3) {
					localStorage.clear()
					this.$router.push({
						name: "DomainList"
					})
				}
				// 切换到其他模块时，将目标域名和解析IP作为路由参数传递
				else if (name >= 0) {
					this.curModule = name
					this.$router.push({
						name: this.pages[this.curModule][1],
						params: {"domain_name": this.targetDomain, "ips": this.ips}
					})
				}
			}
		}
	}
</script>

<style scoped>
.current-target{
	padding: 15px;
	font-size: 18px;
	font-weight: bold;
	background: #fff;
	border-bottom: 1px solid #f5f7f9;
}
.layout{
	border: 1px solid #d7dde4;
	background: #f5f7f9;
	position: relative;
	border-radius: 4px;
	overflow: hidden;
}
.nav-head{
	display: flex;
	flex-direction: row;
	width: 500px;
	height: 60px;
	color: #fff;
}
.nav-head-logo{
	margin: 5px 15px;
	font-size: 24px;
	font-weight: bold;
}
.nav-head-title{
	display: flex;
	flex-direction: column;
	align-items: left;
	font-size: 14px;
}
.nav-head-title div{
	height: 30px;
	margin: -5px 0;
}
.layout-nav{
	width: 420px;
	margin: 0 auto;
	margin-right: 20px;
}
.content{
	padding: 10px 20px 20px 20px;
	min-height: 280px;
	background: #fff;
}
.footer{
	text-align: center;
}
.footer div span{
	font-size: 12px;
	margin: 0 20px;
}
</style>