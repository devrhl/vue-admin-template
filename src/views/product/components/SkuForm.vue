<template>
  <el-form label-width="100px">
    <el-form-item label="SPU 名称">
      <span>{{spu.spuName}}</span>
    </el-form-item>

    <el-form-item label="SKU 名称">
      <el-input type="text" placeholder="SKU 名称" v-model="skuInfo.skuName"></el-input>
    </el-form-item>
    
    <el-form-item label="价格(元)">
      <el-input type="number" placeholder="SKU 价格" v-model="skuInfo.price"></el-input>
    </el-form-item>

    <el-form-item label="重量(千克)">
      <el-input type="number" placeholder="SKU 重量" v-model="skuInfo.weight"></el-input>
    </el-form-item>

    <el-form-item label="规格描述">
      <el-input type="textarea" placeholder="SKU 规格描述" rows="4" v-model="skuInfo.skuDesc"></el-input>
    </el-form-item>

    <el-form-item label="平台属性">
      <el-form inline label-width="100px">
        <el-form-item :label="attr.attrName" style="margin: 5px 0" v-for="attr in attrList" :key="attr.id">
          <el-select placeholder="请输入" v-model="attr.attrIdValueId">
            <el-option :label="value.valueName" :value="`${attr.id}:${value.id}`" v-for="value in attr.attrValueList" :key="value.id"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
    </el-form-item>

    <el-form-item label="销售属性">
      <el-form inline label-width="100px">
        <el-form-item :label="attr.saleAttrName" style="margin: 5px 0" 
          v-for="attr in spuSaleAttrList" :key="attr.id">
          <el-select v-model="attr.saleAttrValueId" placeholder="请输入">
            <el-option :label="value.saleAttrValueName" :value="value.id" 
              v-for="value in attr.spuSaleAttrValueList" :key="value.id" />
          </el-select>
        </el-form-item>
      </el-form>
    </el-form-item>

    <el-form-item label="图片列表">
      <el-table border :data="spuImageList" @selection-change="handleSelectionChange">
        <el-table-column
          type="selection"
          width="55">
        </el-table-column>

        <el-table-column label="图片">
          <template slot-scope="{row}">
            <img :src="row.imgUrl" alt="" style="width:100px;height:100px;">
          </template>
        </el-table-column>
        <el-table-column label="名称" prop="imgName"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="{row, $index}">
            <el-tag type="success" v-if="row.isDefault==='1'">默认</el-tag>
            <el-button type="primary" size="small" v-else @click="setDefault(row)">设为默认</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-form-item>

    <el-form-item>
      <el-button type="primary" @click="save">保存</el-button>
      <el-button @click="back">取消</el-button>
    </el-form-item>
  </el-form>
</template>

<script>
export default {
  name: 'SkuForm',

  props: {
    cancel: Function
  },

  data () {
    return {
      spu: {}, // 所属的spu对象
      skuInfo: {
        // 下面3个数据从父组件传入收集
			  category3Id: null, // 3级分类ID
			  spuId: null, // SPU的id
        tmId: null, // 品牌ID
        
        // 下面4个通过v-model收集
			  skuName: null, // sku的名称
			  skuDesc: null, // sku的描述
			  price: null, // sku的价格
        weight: null, // sku的重量
        
			  skuDefaultImg: null, // sku的默认图片  
			  skuAttrValueList: [], // sku的属性值列表
			  skuSaleAttrValueList: [], // sku属性属性值列表
			  skuImageList: [], // 选择的spu图片列表
			},

      attrList: [], // 平台属性数组
      spuSaleAttrList: [], // spu销售属性数组
      spuImageList: [], // spu图片数组
      selectedSpuImageList: [], // 所有选中的spu图片列表
    }
  },

  methods: {

    /* 
    重置数据
    */
    resetData () {
      this.spu = {}// 所属的spu对象
      this.skuInfo = {
        // 下面3个数据从父组件传入收集
			  category3Id: null, // 3级分类ID
			  spuId: null, // SPU的id
        tmId: null, // 品牌ID
        
        // 下面4个通过v-model收集
			  skuName: null, // sku的名称
			  skuDesc: null, // sku的描述
			  price: null, // sku的价格
        weight: null, // sku的重量
        
			  skuDefaultImg: null, // sku的默认图片  
			  skuAttrValueList: [], // sku的属性值列表
			  skuSaleAttrValueList: [], // sku属性属性值列表
			  skuImageList: [], // 选择的spu图片列表
			}

      this.attrList = [] // 平台属性数组
      this.spuSaleAttrList = [] // spu销售属性数组
      this.spuImageList = [] // spu图片数组
      this.selectedSpuImageList = [] // 所有选中的spu图片列表
    },

    /* 
    取消保存返回列表界面
    */
    // cancel () {  // 不能用这个名称, 与props传入的属性名称重复了
    back () {
      // 重置数据
      this.resetData()

      // 利用函数props向父组件通信
      this.cancel()
    },

    /* 
    保存当前输入收集的SPU信息
    */
    async save () {
      // 取出请求需要的数据
      const {skuInfo, attrList, spuSaleAttrList, selectedSpuImageList} = this
      /* 
      整理1: 平台属性
        目标数据: skuInfo.skuAttrValueList
          {
              "attrId": "2",
              "valueId": "9"
            }
        现有数据: attrList
          {
            attrIdValueId: '2:9' // 可能没有
          }
      */
      attrList.forEach(attr => {
        if (attr.attrIdValueId) {
          const [attrId, valueId] = attr.attrIdValueId.split(':') // ['2', '9']
          skuInfo.skuAttrValueList.push({
            attrId,
            valueId
          })
        }
      })

      /* 
      整理2: 销售属性
        目标数据: spuInfo.skuSaleAttrValueList
          {
              "saleAttrValueId": 258  
            }
        现有数据: spuSaleAttrList
          {
            saleAttrValueId: 258 // 可能没有
          }
      */
      skuInfo.skuSaleAttrValueList = spuSaleAttrList.reduce((pre, attr) => {
        if (attr.saleAttrValueId) {
          pre.push({saleAttrValueId: attr.saleAttrValueId})
        }
        return pre
      }, [])

      /* 
      整理3: 图片列表
        目标数据: spuInfo.skuImageList
          {
            "imgName": "下载 (1).jpg",
            "imgUrl": "http://47.93.148.192:8080/xxx.jpg",
            "spuImgId": 337, // 当前Spu图片的id
            "isDefault": "1"   // 默认为"1", 非默认为"0"
          }
        现有数据: selectedSpuImageList
          {
            "id": 333,
            "spuId": 26,
            "imgName": "rBHu8l6UcKyAfzDsAAAPN5YrVxw870.jpg",
            "imgUrl": "http://47.93.148.192:8080/xxx.jpg",
            "isDefault": "1" / "0"
          }
      */
      skuInfo.skuImageList = selectedSpuImageList.map(item => ({
        imgName: item.imgName,
        imgUrl: item.imgUrl,
        spuImgId: item.id, // 当前Spu图片的id
        isDefault: item.isDefault  // 默认为"1", 非默认为"0"
      }))

      // 发请求
      await this.$API.sku.addUpdate(skuInfo)
      // 成功了...
      this.$message.success('保存SKU成功')
      // 重置数据
      this.resetData()
      // 通知父组件
      this.$emit('success')
      
    },

    /* 
    勾选项的状态发生改变的事件监听回调
    val: 是所有选中行的数据对象的数组
    */
    handleSelectionChange (val) {
      console.log('handleSelectionChange()', val) // 通过打印来查看回调函数的参数的具体情况
      this.selectedSpuImageList = val
    },

    /* 
    将指定图片设置为默认的
    */
    setDefault (image) {
      // 需要将原来默认的变为非默认的 ==>得遍历
      this.spuImageList.forEach(item => item.isDefault = '0')
      // 将当前图片设置为默认的
      image.isDefault = '1'
      // 收集数据(默认图片)
      this.skuInfo.skuDefaultImg = image.imgUrl
    },

    /* 
    初始化请求加载添加界面需要的数据
    */
    initLoadAddData (spu) {
      this.spu = spu

      // 收集父组件传入的数据到skuInfo
      // category3Id: null, // 3级分类ID
      // spuId: null, // SPU的id
      // tmId: null, // 品牌ID
      this.skuInfo.category3Id = spu.category3Id
      this.skuInfo.spuId = spu.id
      this.skuInfo.tmId = spu.tmId

      // 请求获取数据显示
      this.getAddData()
    },

    /* 
    根据3个级别的分类ID获取所有平台属性列表: attr.getList (category1Id, category2Id, category3Id) 
		根据spuId获取SPU图片列表: sku.getSpuImageList(spuId)
    根据spuId获取spu销售属性列表: sku.getSpuSaleAttrList(spuId)
    
    只有当都成功时才更新数据显示  ===>  Promise.all(promises)
    */
    getAddData () {
      // 取出请求条件数据
      const {category1Id, category2Id, category3Id, id} = this.spu

      // 连续发送3个异步请求, 并得到其对应的promise
      const promise1 = this.$API.attr.getList (category1Id, category2Id, category3Id) 
      const promise2 = this.$API.sku.getSpuSaleAttrList(id)
      const promise3 = this.$API.sku.getSpuImageList(id)

      // 只有当都成功时才更新数据显示  ===>  Promise.all(promises)
      Promise.all([promise1, promise2, promise3]).then(results => {
        const attrList = results[0].data
        const spuSaleAttrList = results[1].data
        // 给spuImageList数组中的每个元素添加一个属性: isDefault='0'  开始都是非默认的
        // const spuImageList = results[2].data.map(item => ({...item, isDefault: '0'}))
        const spuImageList = results[2].data.map(item => {
          item.isDefault = '0'
          return item
        })

        // 更新数据
        this.attrList = attrList
        this.spuSaleAttrList = spuSaleAttrList
        this.spuImageList = spuImageList
      })
    }
  }
}
</script>
