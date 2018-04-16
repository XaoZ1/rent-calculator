<template>
	<div id="wrapper">
		<div class="title">租金计算器</div>
		<ul class="input-wrap">
			<li class="item-wrap">
				<label>起止日期:</label>
				<div class="input">
					<el-date-picker
						v-model="value6"
						:picker-options="pickerOptions"
						type="daterange"
						range-separator="至"
						start-placeholder="开始日期"
						end-placeholder="结束日期"
						@change="changeData"
					>
					</el-date-picker>
				</div>
				
			</li>
			<li class="item-wrap">
				<label>月租金:</label>
				<div class="input">
					<el-input v-model="rent" placeholder="请输入月租金"></el-input>
				</div>
			</li>
			<li class="item-wrap">
				<label>递增比率:</label>
				<div class="input">
					<el-input v-model="ratio" placeholder="请输入递增比率"></el-input>
				</div>
			</li>
			<li class="item-wrap">
				<div class="btn" @click="startCalculator()">开始计算</div>
			</li>
		</ul>
		<ul class="result-list">
			<li class="year-list" v-for="(item, index) in returnDataList" :key="index">
				<div class="title">{{item.year}}</div>
				<ul class="month-list">
					<li>
						<label>月份</label>
						<div class="price">月租金</div>
					</li>
					<li v-for="(f, i) in item.monthData" :key="i">
						<label>{{f.month}}</label>
						<div class="price">{{f.price}}</div>
					</li>
				</ul>
				<div>总计：{{item.totalPrice}}</div>
			</li>
			<li>总租金：{{allTotalPrice}}</li>
		</ul>
	</div>
</template>

<script>

  export default {
    data () {
		return {
			value6: '',
			pickerOptions: {
				onPick (maxDate, minDate) {
					// console.log(maxDate)
					// console.log(minDate)
				}
			},
			timeData: {},
			startTimeData: '', // 开始时间
			endTimeData: '', // 结束时间
			startTimeObj: {
				year: '',
				month: '',
				day: ''
			},
			endTimeObj: {
				year: '',
				month: '',
				day: ''
			},
			rent: 0, // 租金
			ratio: 1, // 比率
			m: 0, // 相差月份
			returnDataList: [], // 处理好的租金数据
			allTotalPrice: 0
		}
    },
    methods: {
		startCalculator () {
			this.calculator()
		},
		test (date1, date2) {
			date1 = date1.split('-');
			// 得到月数
			date1 = parseInt(date1[0]) * 12 + parseInt(date1[1]);
			// 拆分年月日
			date2 = date2.split('-');
			// 得到月数
			date2 = parseInt(date2[0]) * 12 + parseInt(date2[1]);
			this.m = Math.abs(date1 - date2);
			// console.log(this.m)
			
			// alert(m);
		},
		calculator () {
			this.returnDataList = []
			this.allTotalPrice = []
			if (this.startTimeObj.year == this.endTimeObj.year) { // 只有一年的情况
				this.returnDataList.push({
					year: this.startTimeObj.year,
					monthData: []
				})
				// console.log(1111)
				if (this.m != 0) { // 一年中多月份的情况
					console.log(this.startTimeObj.month)
					let arr = []
					for (let i = this.startTimeObj.month; i < this.m + this.startTimeObj.month+1; i++) {
						console.log(i)
						arr.push(i)
					}
					if (arr.length == 2) {  // 一年两个月的情况
						this.oneYearTwoMonth(0)
						
					} else if (arr.length > 2){ // 一年两个月以上的情况
						this.oneYearLotsMonth(arr, 0)
					}
					// console.log(arr)
				} else {  // 只有一年一个月的情况
					this.oneYearOneMonth(0)	
				}
				
			} else {
				
				let arr = []
				let ratioList = [] // 倍率列表
				let yearRatioList = [] // 倍率年份列表
				let lastRatioFlag = 0 // 上次倍率标记
				let ratioFlag = 1 // 倍率标记
				let yearArr = []
				let startMonth = this.startTimeObj.month
				for (let o = 0; o <= this.m; o++) {
					// console.log(o)
					if (startMonth > 12) {
						startMonth = 1
					}
					if (o / 12 <= ratioFlag && o / 12>= lastRatioFlag) {
						ratioList.push(ratioFlag -1)
						if (o / 12 == ratioFlag) {
							ratioFlag ++
							lastRatioFlag ++
						}
					}
					arr.push(startMonth)
					startMonth++
				}
				let flag = 0
				arr.forEach((item, index) => {
					if (item == 12) {
						yearArr.push(arr.slice(flag, index+1))
						yearRatioList.push(ratioList.slice(flag, index+1))
						flag = index +1
					}
					
				})
				if (flag < arr.length) {
					yearArr.push(arr.slice(flag, arr.length))
					yearRatioList.push(ratioList.slice(flag, ratioList.length))
				}
				// console.log(arr)
				// console.log(yearArr)
				// console.log(ratioList)
				// console.log(yearRatioList)
				for (let j = 0; j <this.endTimeObj.year - this.startTimeObj.year+1; j++) {
					this.returnDataList.push({
						year: this.startTimeObj.year + j,
						monthData: []
					})
					// console.log(yearArr[j].length)
					// console.log(j)
					if (yearArr[j].length == 1) {
						this.oneYearOneMonth(j, yearArr)
						// console.log("1222")
					} else if (yearArr[j].length == 2) {
						this.oneYearTwoMonth(j, true, yearArr, yearArr[j])
					} else if (yearArr[j].length > 2) {
						this.oneYearLotsMonth(yearArr[j], j, true, yearArr, yearRatioList[j])
					}
				}
				
				// if () {

				// }
			}
			console.log(this.returnDataList)
			this.returnDataList.forEach((item, index) => {
				this.allTotalPrice = Number(item.totalPrice) + Number(this.allTotalPrice)
			})
			
		},
		oneYearLotsMonth (arr, index, isLots, yearArr, ratioArr) { //一年多月
			if (isLots) {
				if (index == 0) {
					let isRun = (this.startTimeObj.year % 4 == 0) && (this.startTimeObj.year % 100 != 0 || this.startTimeObj.year % 400 == 0) 
					if (this.startTimeObj.month == 1 ||
						this.startTimeObj.month == 3 || 
						this.startTimeObj.month == 5 ||
						this.startTimeObj.month == 7 ||
						this.startTimeObj.month == 8 ||
						this.startTimeObj.month == 10 ||
						this.startTimeObj.month == 12 ) {
							if (this.startTimeObj.day == 1
							) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent),
									month: this.startTimeObj.month
								})
							} else {
								this.returnDataList[index].monthData.push({
									price: (31 - this.startTimeObj.day) / 30 * (Number(this.rent)),
									month: this.startTimeObj.month
								})
							}

					} else if (this.startTimeObj.month == 4 || 
						this.startTimeObj.month == 6 || 
						this.startTimeObj.month == 9 ||
						this.startTimeObj.month == 11) {
							if (this.startTimeObj.day == 1) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent),
									month: this.startTimeObj.month
								})
							} else {
								this.returnDataList[index].monthData.push({
									price: (30 - this.startTimeObj.day) / 30 * (Number(this.rent)),
									month: this.startTimeObj.month
								})
							}
					} else {
						if ((this.startTimeObj.year % 4 == 0) && (this.startTimeObj.year % 100 != 0 || this.startTimeObj.year % 400 == 0)) {
							if (this.startTimeObj.day == 1 ) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent),
									month: this.startTimeObj.month
								})
							} else {
								this.returnDataList[index].monthData.push({
									price: (29 - this.startTimeObj.day) / 30 * (Number(this.rent)),
									month: this.startTimeObj.month
								})
							}
						} else {
							if (this.startTimeObj.day == 1) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent),
									month: this.startTimeObj.month
								})
							} else {
								this.returnDataList[index].monthData.push({
									price: (28 - this.startTimeObj.day) / 30 * (Number(this.rent)),
									month: this.startTimeObj.month
								})
							}
						}
					}
					console.log("hahahahahah")
					for (let k =1; k < arr.length; k++) {
						// console.log("哈哈哈")
						// console.log(arr[k])
						this.returnDataList[index].monthData.push({
							price: Number(this.rent),
							month: arr[k]
						})
					}
				} else if (index == yearArr.length - 1) {
					let isRun = (this.endTimeObj.year % 4 == 0) && (this.endTimeObj.year % 100 != 0 || this.endTimeObj.year % 400 == 0)
					for (let k =0; k < arr.length-1; k++) {
						// console.log("哈哈哈")
						// console.log(arr[k])
						this.returnDataList[index].monthData.push({
							price: Number(this.rent) * (ratioArr[ratioArr.length-1] == 0 ? 1 : Math.pow(this.ratio, ratioArr[ratioArr.length-1])),
							month: arr[k]
						})
					}
					if (this.endTimeObj.month == 1 ||
						this.endTimeObj.month == 3 || 
						this.endTimeObj.month == 5 ||
						this.endTimeObj.month == 7 ||
						this.endTimeObj.month == 8 ||
						this.endTimeObj.month == 10 ||
						this.endTimeObj.month == 12 ) {
							if (this.endTimeObj.day == 31) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent) * (ratioArr[ratioArr.length-1] == 0 ? 1 : Math.pow(this.ratio, ratioArr[ratioArr.length-1])),
									month: this.endTimeObj.month
								})
							}  else {
								this.returnDataList[index].monthData.push({
									price: (30 - (31 - this.endTimeObj.day)) / 30 *(Number(this.rent)) * (ratioArr[ratioArr.length-1] == 0 ? 1 : Math.pow(this.ratio, ratioArr[ratioArr.length-1])),
									month: this.endTimeObj.month
								})
							}

					} else if (this.endTimeObj.month == 4 || 
						this.endTimeObj.month == 6 || 
						this.endTimeObj.month == 9 ||
						this.endTimeObj.month == 11) {
							if (this.endTimeObj.day == 30) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent) * (ratioArr[ratioArr.length-1] == 0 ? 1 : Math.pow(this.ratio, ratioArr[ratioArr.length-1])),
									month: this.endTimeObj.month
								})
							}  else {
								this.returnDataList[index].monthData.push({
									price: (30 - (30 - this.endTimeObj.day)) / 30 *(Number(this.rent)) * (ratioArr[ratioArr.length-1] == 0 ? 1 : Math.pow(this.ratio, ratioArr[ratioArr.length-1])),
									month: this.endTimeObj.month
								})
							}
					} else {
						if ((this.endTimeObj.year % 4 == 0) && (this.endTimeObj.year % 100 != 0 || this.endTimeObj.year % 400 == 0)) {
							if (this.endTimeObj.day == 29) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent) * (ratioArr[ratioArr.length-1] == 0 ? 1 : Math.pow(this.ratio, ratioArr[ratioArr.length-1])),
									month: this.endTimeObj.month
								})
							} else {
								this.returnDataList[index].monthData.push({
									price: (30 - (29 - this.endTimeObj.day)) / 30 * (Number(this.rent)) * (ratioArr[ratioArr.length-1] == 0 ? 1 : Math.pow(this.ratio, ratioArr[ratioArr.length-1])),
									month: this.endTimeObj.month
								})
							}
						} else {
							if (this.endTimeObj.day == 28) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent) * (ratioArr[ratioArr.length-1] == 0 ? 1 : Math.pow(this.ratio, ratioArr[ratioArr.length-1])),
									month: this.endTimeObj.month
								})
							} else {
								this.returnDataList[index].monthData.push({
									price: (30 - (28 - this.endTimeObj.day)) / 30 * (Number(this.rent)) * (ratioArr[ratioArr.length-1] == 0 ? 1 : Math.pow(this.ratio, ratioArr[ratioArr.length-1])),
									month: this.endTimeObj.month
								})
							}
						}
					}
					
				} else {
					for (let k =0; k < arr.length; k++) {
						// console.log("哈哈哈")
						// console.log(arr[k])
						// console.log(k)
						console.log(ratioArr[k])

						this.returnDataList[index].monthData.push({
							price: Number(this.rent) * (ratioArr[k] == 0 ? 1 : Math.pow(this.ratio, ratioArr[k])),
							month: arr[k]
						})
					}
				}
				
				
			} else {
				let isRun = (this.startTimeObj.year % 4 == 0) && (this.startTimeObj.year % 100 != 0 || this.startTimeObj.year % 400 == 0) 
				if (this.startTimeObj.month == 1 ||
					this.startTimeObj.month == 3 || 
					this.startTimeObj.month == 5 ||
					this.startTimeObj.month == 7 ||
					this.startTimeObj.month == 8 ||
					this.startTimeObj.month == 10 ||
					this.startTimeObj.month == 12 ) {
						if (this.startTimeObj.day == 1 && this.endTimeObj.day == 29 && this.endTimeObj.month == 2 && isRun == true||
							this.startTimeObj.day == 1 && this.endTimeObj.day == 28 && this.endTimeObj.month == 2 && isRun == false
						) {
							this.returnDataList[index].monthData.push({
								price: Number(this.rent),
								month: this.startTimeObj.month
							}, {
								price: Number(this.rent),
								month: this.endTimeObj.month
							})
						} else if (this.startTimeObj.day == 1 && this.endTimeObj.day == 30 ||
						this.startTimeObj.day == 1 && this.endTimeObj.day == 31 || this.endTimeObj.month == 8) {
							this.returnDataList[index].monthData.push({
								price: Number(this.rent),
								month: this.startTimeObj.month
							}, {
								price: Number(this.rent),
								month: this.endTimeObj.month
							})
						}else {
							this.returnDataList[index].monthData.push({
								price: (31 - this.startTimeObj.day) / 30 * (Number(this.rent)),
								month: this.startTimeObj.month
							}, {
								price: (30 - (31 - this.endTimeObj.day)) / 30 *(Number(this.rent)),
								month: this.endTimeObj.month
							})
						}

				} else if (this.startTimeObj.month == 4 || 
					this.startTimeObj.month == 6 || 
					this.startTimeObj.month == 9 ||
					this.startTimeObj.month == 11) {
						if (this.startTimeObj.day == 1 && this.endTimeObj.day == 31) {
							this.returnDataList[index].monthData.push({
								price: Number(this.rent),
								month: this.startTimeObj.month
							}, {
								price: Number(this.rent),
								month: this.endTimeObj.month
							})
						} else if (this.startTimeObj.day == 1 && this.endTimeObj.day != 31) {
							this.returnDataList[index].monthData.push({
								price: Number(this.rent),
								month: this.startTimeObj.month
							}, {
								price: (30 - (30 - this.endTimeObj.day)) / 30 *(Number(this.rent)),
								month: this.endTimeObj.month
							})
						} else if (this.startTimeObj.day != 1 && this.endTimeObj.day == 31) {
							this.returnDataList[index].monthData.push({
								price: (30 - this.startTimeObj.day) / 30 * (Number(this.rent)),
								month: this.startTimeObj.month
							}, {
								price: Number(this.rent),
								month: this.endTimeObj.month
							})
						} else {
							this.returnDataList[index].monthData.push({
								price: (30 - this.startTimeObj.day) / 30 * (Number(this.rent)),
								month: this.startTimeObj.month
							}, {
								price: (30 - (30 - this.endTimeObj.day)) / 30 *(Number(this.rent)),
								month: this.endTimeObj.month
							})
						}
				} else {
					if ((this.startTimeObj.year % 4 == 0) && (this.startTimeObj.year % 100 != 0 || this.startTimeObj.year % 400 == 0)) {
						if (this.startTimeObj.day == 1 && this.endTimeObj.day == 31) {
							this.returnDataList[index].monthData.push({
								price: Number(this.rent),
								month: this.startTimeObj.month
							}, {
								price: Number(this.rent),
								month: this.endTimeObj.month
							})
						} else if (this.startTimeObj.day == 1 && this.endTimeObj.day != 31) {
							this.returnDataList[index].monthData.push({
								price: Number(this.rent),
								month: this.startTimeObj.month
							}, {
								price: (30 - (29 - this.endTimeObj.day)) / 30 *(Number(this.rent)),
								month: this.endTimeObj.month
							})
						} else if (this.startTimeObj.day != 1 && this.endTimeObj.day == 31) {
							this.returnDataList[index].monthData.push({
								price: (29 - this.startTimeObj.day) / 30 * (Number(this.rent)),
								month: this.startTimeObj.month
							}, {
								price: Number(this.rent),
								month: this.endTimeObj.month
							})
						} else {
							this.returnDataList[index].monthData.push({
								price: (29 - this.startTimeObj.day) / 30 * (Number(this.rent)),
								month: this.startTimeObj.month
							}, {
								price: (30 - (29 - this.endTimeObj.day)) / 30 *(Number(this.rent)),
								month: this.endTimeObj.month
							})
						}
					} else {
						if (this.startTimeObj.day == 1 && this.endTimeObj.day == 31) {
							this.returnDataList[index].monthData.push({
								price: Number(this.rent),
								month: this.startTimeObj.month
							}, {
								price: Number(this.rent),
								month: this.endTimeObj.month
							})
						} else {
							this.returnDataList[index].monthData.push({
								price: (28 - this.startTimeObj.day) / 30 * (Number(this.rent)),
								month: this.startTimeObj.month
							}, {
								price: (30 - ( - this.endTimeObj.day)) / 30 * (Number(this.rent)),
								month: this.endTimeObj.month
							})
						}
					}
				}
				for (let k =1; k < arr.length-1; k++) {
					// console.log("哈哈哈")
					console.log(arr[k])
					this.returnDataList[index].monthData.push({
						price: Number(this.rent),
						month: arr[k]
					})
				}
			}
			
			// console.log(this.returnDataList)
			let totalPrice = 0
			this.returnDataList[index].monthData.forEach((item, index) => {
				totalPrice += Number(item.price)
			})
			this.returnDataList[index].totalPrice = totalPrice
			// console.log(this.returnDataList)
		},
		oneYearTwoMonth (index, isLots, arr1, arr) { // 一年两月
			if (isLots) {
				if (index == 0) {
					let isRun = (this.startTimeObj.year % 4 == 0) && (this.startTimeObj.year % 100 != 0 || this.startTimeObj.year % 400 == 0) 
					if (this.startTimeObj.month == 1 ||
						this.startTimeObj.month == 3 || 
						this.startTimeObj.month == 5 ||
						this.startTimeObj.month == 7 ||
						this.startTimeObj.month == 8 ||
						this.startTimeObj.month == 10 ||
						this.startTimeObj.month == 12 ) {
							if (this.startTimeObj.day == 1
							) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent),
									month: this.startTimeObj.month
								})
							} else {
								this.returnDataList[index].monthData.push({
									price: (31 - this.startTimeObj.day) / 30 * (Number(this.rent)),
									month: this.startTimeObj.month
								})
							}

					} else if (this.startTimeObj.month == 4 || 
						this.startTimeObj.month == 6 || 
						this.startTimeObj.month == 9 ||
						this.startTimeObj.month == 11) {
							if (this.startTimeObj.day == 1) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent),
									month: this.startTimeObj.month
								})
							} else {
								this.returnDataList[index].monthData.push({
									price: (30 - this.startTimeObj.day) / 30 * (Number(this.rent)),
									month: this.startTimeObj.month
								})
							}
					} else {
						if ((this.startTimeObj.year % 4 == 0) && (this.startTimeObj.year % 100 != 0 || this.startTimeObj.year % 400 == 0)) {
							if (this.startTimeObj.day == 1 ) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent),
									month: this.startTimeObj.month
								})
							} else {
								this.returnDataList[index].monthData.push({
									price: (29 - this.startTimeObj.day) / 30 * (Number(this.rent)),
									month: this.startTimeObj.month
								})
							}
						} else {
							if (this.startTimeObj.day == 1) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent),
									month: this.startTimeObj.month
								})
							} else {
								this.returnDataList[index].monthData.push({
									price: (28 - this.startTimeObj.day) / 30 * (Number(this.rent)),
									month: this.startTimeObj.month
								})
							}
						}
					}
					this.returnDataList[index].monthData.push({
						price: Number(this.rent),
						month: arr[1]
					})

				} else if (index == 1) {
					console.log(arr)
					let isRun = (this.endTimeObj.year % 4 == 0) && (this.endTimeObj.year % 100 != 0 || this.endTimeObj.year % 400 == 0)
					this.returnDataList[index].monthData.push({
						price: Number(this.rent),
						month: arr[0]
					})
					if (this.endTimeObj.month == 1 ||
						this.endTimeObj.month == 3 || 
						this.endTimeObj.month == 5 ||
						this.endTimeObj.month == 7 ||
						this.endTimeObj.month == 8 ||
						this.endTimeObj.month == 10 ||
						this.endTimeObj.month == 12 ) {
							if (this.endTimeObj.day == 31) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent),
									month: this.endTimeObj.month
								})
							}  else {
								this.returnDataList[index].monthData.push({
									price: (30 - (31 - this.endTimeObj.day)) / 30 *(Number(this.rent)),
									month: this.endTimeObj.month
								})
							}

					} else if (this.endTimeObj.month == 4 || 
						this.endTimeObj.month == 6 || 
						this.endTimeObj.month == 9 ||
						this.endTimeObj.month == 11) {
							if (this.endTimeObj.day == 30) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent),
									month: this.endTimeObj.month
								})
							}  else {
								this.returnDataList[index].monthData.push({
									price: (30 - (30 - this.endTimeObj.day)) / 30 *(Number(this.rent)),
									month: this.endTimeObj.month
								})
							}
					} else {
						if ((this.endTimeObj.year % 4 == 0) && (this.endTimeObj.year % 100 != 0 || this.endTimeObj.year % 400 == 0)) {
							if (this.endTimeObj.day == 29) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent),
									month: this.endTimeObj.month
								})
							} else {
								this.returnDataList[index].monthData.push({
									price: (30 - (29 - this.endTimeObj.day)) / 30 * (Number(this.rent)),
									month: this.endTimeObj.month
								})
							}
						} else {
							if (this.endTimeObj.day == 28) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent),
									month: this.endTimeObj.month
								})
							} else {
								this.returnDataList[index].monthData.push({
									price: (30 - (28 - this.endTimeObj.day)) / 30 * (Number(this.rent)),
									month: this.endTimeObj.month
								})
							}
						}
					}
					
					
				}
				
			} else {
				let isRun = (this.startTimeObj.year % 4 == 0) && (this.startTimeObj.year % 100 != 0 || this.startTimeObj.year % 400 == 0) 
				if (this.startTimeObj.month == 1 ||
					this.startTimeObj.month == 3 || 
					this.startTimeObj.month == 5 ||
					this.startTimeObj.month == 7 ||
					this.startTimeObj.month == 8 ||
					this.startTimeObj.month == 10 ||
					this.startTimeObj.month == 12 ) {
						if (this.startTimeObj.day == 1 && this.endTimeObj.day == 29 && this.endTimeObj.month == 2 && isRun == true||
							this.startTimeObj.day == 1 && this.endTimeObj.day == 28 && this.endTimeObj.month == 2 && isRun == false
						) {
							this.returnDataList[index].monthData.push({
								price: Number(this.rent),
								month: this.startTimeObj.month
							}, {
								price: Number(this.rent),
								month: this.endTimeObj.month
							})
						} else if (this.startTimeObj.day == 1 && this.endTimeObj.day == 30 ||
						this.startTimeObj.day == 1 && this.endTimeObj.day == 31 || this.endTimeObj.month == 8) {
							this.returnDataList[index].monthData.push({
								price: Number(this.rent),
								month: this.startTimeObj.month
							}, {
								price: Number(this.rent),
								month: this.endTimeObj.month
							})
						}else {
							this.returnDataList[index].monthData.push({
								price: (31 - this.startTimeObj.day) / 30 * (Number(this.rent)),
								month: this.startTimeObj.month
							}, {
								price: (30 - (31 - this.endTimeObj.day)) / 30 *(Number(this.rent)),
								month: this.endTimeObj.month
							})
						}

				} else if (this.startTimeObj.month == 4 || 
					this.startTimeObj.month == 6 || 
					this.startTimeObj.month == 9 ||
					this.startTimeObj.month == 11) {
						if (this.startTimeObj.day == 1 && this.endTimeObj.day == 31) {
							this.returnDataList[index].monthData.push({
								price: Number(this.rent),
								month: this.startTimeObj.month
							}, {
								price: Number(this.rent),
								month: this.endTimeObj.month
							})
						} else if (this.startTimeObj.day == 1 && this.endTimeObj.day != 31) {
							this.returnDataList[index].monthData.push({
								price: Number(this.rent),
								month: this.startTimeObj.month
							}, {
								price: (30 - (30 - this.endTimeObj.day)) / 30 *(Number(this.rent)),
								month: this.endTimeObj.month
							})
						} else if (this.startTimeObj.day != 1 && this.endTimeObj.day == 31) {
							this.returnDataList[index].monthData.push({
								price: (30 - this.startTimeObj.day) / 30 * (Number(this.rent)),
								month: this.startTimeObj.month
							}, {
								price: Number(this.rent),
								month: this.endTimeObj.month
							})
						} else {
							this.returnDataList[index].monthData.push({
								price: (30 - this.startTimeObj.day) / 30 * (Number(this.rent)),
								month: this.startTimeObj.month
							}, {
								price: (30 - (30 - this.endTimeObj.day)) / 30 *(Number(this.rent)),
								month: this.endTimeObj.month
							})
						}
				} else {
					if ((this.startTimeObj.year % 4 == 0) && (this.startTimeObj.year % 100 != 0 || this.startTimeObj.year % 400 == 0)) {
						if (this.startTimeObj.day == 1 && this.endTimeObj.day == 31) {
							this.returnDataList[index].monthData.push({
								price: Number(this.rent),
								month: this.startTimeObj.month
							}, {
								price: Number(this.rent),
								month: this.endTimeObj.month
							})
						} else if (this.startTimeObj.day == 1 && this.endTimeObj.day != 31) {
							this.returnDataList[index].monthData.push({
								price: Number(this.rent),
								month: this.startTimeObj.month
							}, {
								price: (30 - (29 - this.endTimeObj.day)) / 30 *(Number(this.rent)),
								month: this.endTimeObj.month
							})
						} else if (this.startTimeObj.day != 1 && this.endTimeObj.day == 31) {
							this.returnDataList[index].monthData.push({
								price: (29 - this.startTimeObj.day) / 30 * (Number(this.rent)),
								month: this.startTimeObj.month
							}, {
								price: Number(this.rent),
								month: this.endTimeObj.month
							})
						} else {
							this.returnDataList[index].monthData.push({
								price: (29 - this.startTimeObj.day) / 30 * (Number(this.rent)),
								month: this.startTimeObj.month
							}, {
								price: (30 - (29 - this.endTimeObj.day)) / 30 *(Number(this.rent)),
								month: this.endTimeObj.month
							})
						}
					} else {
						if (this.startTimeObj.day == 1 && this.endTimeObj.day == 31) {
							this.returnDataList[index].monthData.push({
								price: Number(this.rent),
								month: this.startTimeObj.month
							}, {
								price: Number(this.rent),
								month: this.endTimeObj.month
							})
						} else {
							this.returnDataList[index].monthData.push({
								price: (28 - this.startTimeObj.day) / 30 * (Number(this.rent)),
								month: this.startTimeObj.month
							}, {
								price: (30 - ( - this.endTimeObj.day)) / 30 *(Number(this.rent)),
								month: this.endTimeObj.month
							})
						}
					}
				}
			}
			
			let totalPrice = 0
			this.returnDataList[index].monthData.forEach((item, index) => {
				totalPrice += Number(item.price)
			})
			this.returnDataList[index].totalPrice = totalPrice
		},
		oneYearOneMonth (index, yArr) { // 一年一月
			if (yArr) {
				if (index == 0) {
					if (this.startTimeObj.month == 1 ||
						this.startTimeObj.month == 3 || 
						this.startTimeObj.month == 5 ||
						this.startTimeObj.month == 7 ||
						this.startTimeObj.month == 8 ||
						this.startTimeObj.month == 10 ||
						this.startTimeObj.month == 12 ) {
						if (this.startTimeObj.day == 1) {
							this.returnDataList[index].monthData.push({
								price: Number(this.rent),
								month: this.startTimeObj.month
							})
						} else {
							this.returnDataList[index].monthData.push({
								price: (31 - this.startTimeObj.day) / 30 * (Number(this.rent)),
								month: this.startTimeObj.month
							})
						}
						
					} else if (this.startTimeObj.month == 4 || 
						this.startTimeObj.month == 6 || 
						this.startTimeObj.month == 9 ||
						this.startTimeObj.month == 11) {
							if (this.startTimeObj.day == 1) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent),
									month: this.startTimeObj.month
								})
							} else {
								this.returnDataList[index].monthData.push({
									price: (30 - this.startTimeObj.day) / 30 * (Number(this.rent)),
									month: this.startTimeObj.month
								})
							}
					} else {
						if ((this.startTimeObj.year % 4 == 0) && (this.startTimeObj.year % 100 != 0 || this.startTimeObj.year % 400 == 0)) {
							if (this.startTimeObj.day == 1 ) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent),
									month: this.startTimeObj.month
								})
							} else {
								this.returnDataList[index].monthData.push({
									price: (29 - this.startTimeObj.day) / 30 * (Number(this.rent)),
									month: this.startTimeObj.month
								})
							}
						} else {
							if (this.startTimeObj.day == 1) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent),
									month: this.startTimeObj.month
								})
							} else {
								this.returnDataList[index].monthData.push({
									price: (28 - this.startTimeObj.day) / 30 * (Number(this.rent)),
									month: this.startTimeObj.month
								})
							}
						}
					}
				} else {
					if (this.endTimeObj.month == 1 ||
						this.endTimeObj.month == 3 || 
						this.endTimeObj.month == 5 ||
						this.endTimeObj.month == 7 ||
						this.endTimeObj.month == 8 ||
						this.endTimeObj.month == 10 ||
						this.endTimeObj.month == 12 ) {
						if (this.endTimeObj.day == 31) {
							this.returnDataList[index].monthData.push({
								price: Number(this.rent),
								month: this.endTimeObj.month
							})
						} else {
							this.returnDataList[index].monthData.push({
								price: (30 - (31 - this.endTimeObj.day)) / 30 *(Number(this.rent)),
								month: this.endTimeObj.month
							})
						}
						
					} else if (this.endTimeObj.month == 4 || 
						this.endTimeObj.month == 6 || 
						this.endTimeObj.month == 9 ||
						this.endTimeObj.month == 11) {
							if (this.endTimeObj.day == 30) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent),
									month: this.endTimeObj.month
								})
							} else {
								this.returnDataList[index].monthData.push({
									price: (30 - (30 - this.endTimeObj.day)) / 30 *(Number(this.rent)),
									month: this.endTimeObj.month
								})
							}
					} else {
						if ((this.endTimeObj.year % 4 == 0) && (this.endTimeObj.year % 100 != 0 || this.endTimeObj.year % 400 == 0)) {
							if (this.endTimeObj.day == 29) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent),
									month: this.endTimeObj.month
								})
							} else {
								this.returnDataList[index].monthData.push({
									price: (30 - (29 - this.endTimeObj.day)) / 30 *(Number(this.rent)),
									month: this.endTimeObj.month
								})
							}
						} else {
							if (this.endTimeObj.day == 28) {
								this.returnDataList[index].monthData.push({
									price: Number(this.rent),
									month: this.endTimeObj.month
								})
							} else {
								this.returnDataList[index].monthData.push({
									price: (30 - (28 - this.endTimeObj.day)) / 30 *(Number(this.rent)),
									month: this.endTimeObj.month
								})
							}
						}
					}
				}
			} else {
				if (this.startTimeObj.month == 1 ||
					this.startTimeObj.month == 3 || 
					this.startTimeObj.month == 5 ||
					this.startTimeObj.month == 7 ||
					this.startTimeObj.month == 8 ||
					this.startTimeObj.month == 10 ||
					this.startTimeObj.month == 12 ) {
					if (this.startTimeObj.day == 1 && this.endTimeObj.day == 31) {
						this.returnDataList[index].monthData.push({
							price: Number(this.rent),
							month: this.startTimeObj.month
						})
					} else {
						this.returnDataList[index].monthData.push({
							price: (31 - this.startTimeObj.day) / 30 * (Number(this.rent)),
							month: this.startTimeObj.month
						})
					}
					
				} else if (this.startTimeObj.month == 4 || 
					this.startTimeObj.month == 6 || 
					this.startTimeObj.month == 9 ||
					this.startTimeObj.month == 11) {
						if (this.startTimeObj.day == 1 && this.endTimeObj.day == 30) {
							this.returnDataList[index].monthData.push({
								price: Number(this.rent),
								month: this.startTimeObj.month
							})
						} else {
							this.returnDataList[index].monthData.push({
								price: (30 - this.startTimeObj.day) / 30 * (Number(this.rent)),
								month: this.startTimeObj.month
							})
						}
				} else {
					if ((this.startTimeObj.year % 4 == 0) && (this.startTimeObj.year % 100 != 0 || this.startTimeObj.year % 400 == 0)) {
						if (this.startTimeObj.day == 1 && this.endTimeObj.day == 29) {
							this.returnDataList[index].monthData.push({
								price: Number(this.rent),
								month: this.startTimeObj.month
							})
						} else {
							this.returnDataList[index].monthData.push({
								price: (29 - this.startTimeObj.day) / 30 * (Number(this.rent)),
								month: this.startTimeObj.month
							})
						}
					} else {
						if (this.startTimeObj.day == 1 && this.endTimeObj.day == 28) {
							this.returnDataList[index].monthData.push({
								price: Number(this.rent),
								month: this.startTimeObj.month
							})
						} else {
							this.returnDataList[index].monthData.push({
								price: (28 - this.startTimeObj.day) / 30 * (Number(this.rent)),
								month: this.startTimeObj.month
							})
						}
					}
				}
			}
			
			
			this.returnDataList[index].totalPrice = this.returnDataList[index].monthData[0].price
		},
		changeData (val) { // 选择日期时间
			console.log(val)
			this.timeData = val
			this.startTimeData = this.formatTime(val[0], 1)
			this.endTimeData = this.formatTime(val[1], 2)
			// console.log(this.startTimeObj)
			// console.log(this.endTimeObj)
			
			this.test(this.startTimeData, this.endTimeData)
		},
		formatTime ( datetime , sign) {   // 日期格式化方法
			var year = datetime.getFullYear(),  
			month = datetime.getMonth()+1,  
			date = datetime.getDate(),  
			hour = datetime.getHours(),  
			minutes = datetime.getMinutes(),  
			second = datetime.getSeconds();

			if (sign == 1) {
				this.startTimeObj.year = year
				this.startTimeObj.month = month
				this.startTimeObj.day = date
			} else {
				this.endTimeObj.year = year
				this.endTimeObj.month = month
				this.endTimeObj.day = date
			}

			if ( month < 10 ) {  
				month = "0" + month;  
			}  
			if ( date < 10 ) {  
				date = "0" + date;  
			}  
				if ( hour < 10 ) {  
				hour = "0" + hour;  
			}  
			if ( minutes < 10 ) {  
				minutes = "0" + minutes;  
			}  
			if ( second < 10 ) {  
				second = "0" + second;  
			}  
			return (year+"-"+month+"-"+date);  
		}  
    }
  }
</script>

<style lang="scss" scoped>

  * {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  body { font-family: 'Source Sans Pro', sans-serif; }
	#wrapper {
		width: 800px;
		height: 800px;
		margin: 0 auto;
		.title {
			text-align: center;
			margin-bottom: 50px;
			font-size: 30px;
			font-weight: bold;
		}
		.input-wrap {
			.item-wrap {
				height: 50px;
				margin-bottom: 20px;
				label {
					float: left;
					width: 100px;
					height: 50px;
				}
				.input {
					width: 200px;
					float: left;
				}
				.btn {
					width: 300px;
					height: 50px;
					cursor: pointer;
					background:#2993f8;
					line-height: 50px;
					text-align: center;
					color:#fff;
					margin: 0 auto;
				}
			}
		}
		.result-list {
			.year-list {
				border-top: 1px solid #999;
				border-bottom: 1px solid #999;
				list-style: none;
				.title {
					margin-bottom: 10px;
				}
				.month-list {
					padding-left: 50px;
					&>li {
						list-style: none;
						line-height: 30px;
						label {
							width: 200px;
							display: inline-block;
							text-align: center;
							// float: left;
						}
						.price {
							width: 300px;
							text-align: center;
							// float: left;
							display: inline-block;
						}
					}
				}
			}
		}
	}
</style>
