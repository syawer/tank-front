<template>
  <div class="folder-tree">
    <div class="h50 cursor folder-block" :class="{'bg-silver-white': targetMatter.uuid === matter.uuid}"
         @click.stop.prevent="clickItem">
      <span class="fa fa-chevron-down mr5 w14" v-if="pager.data.length && showSubFolder"></span>
      <span class="fa fa-chevron-right mr5 w14" v-if="pager.data.length && !showSubFolder"></span>
      <span :class="{ 'ml23': !pager.data.length }">
				<span v-if="matter.uuid">
					<img :src="matter.getIcon()" class="mr5" alt="文件夹" width="22"/>
					<span>{{matter.name}}</span>
				</span>
				<span v-else>
					<span>根目录</span>
				</span>

			</span>

    </div>
    <NbExpanding>
      <div v-if="pager.data.length && showSubFolder" class="pl20">
        <div v-for="(child, index) in pager.data">
          <FolderTree :matter="child" :targetMatter="targetMatter" :userUuid="userUuid"
                      :callback="callback" :showSubFolderInit="false"/>
        </div>
      </div>
    </NbExpanding>

  </div>
</template>

<script>
  import NbExpanding from '../../../common/widget/NbExpanding'
  import Pager from '../../../common/model/base/Pager'
  import Matter from '../../../common/model/matter/Matter'

  export default {

    name: "FolderTree",
    data() {
      return {
        showSubFolder: false,
        pager: new Pager(Matter)
      }
    },
    props: {
      targetMatter: {
        type: Matter,
        required: true
      },
      matter: {
        type: Matter,
        required: true
      },
      showSubFolderInit: {
        type: Boolean,
        required: false,
        default: false
      },
      userUuid: {
        type: String,
        required: true
      },
      //选择了一个文件夹后回掉，参数matter
      callback: {
        type: Function,
        required: true
      }
    },
    watch: {
      //有可能外面世界的userUuid发生了变化
      'userUuid'(newVal, oldVal) {
        this.refresh()
      }
    },
    methods: {
      clickItem() {
        this.showSubFolder = !this.showSubFolder
        if (this.targetMatter.uuid !== this.matter.uuid) {
          this.targetMatter.render(this.matter)

          if (typeof this.callback === "function") {
            this.callback(this.targetMatter)
          }

        }
      },
      refresh() {

        this.showSubFolder = this.showSubFolderInit

        if (!this.matter.uuid) {
          this.pager.setFilterValue('puuid', 'root')
        } else {
          this.pager.setFilterValue('puuid', this.matter.uuid)
        }

        //限制选择的范围。文件和目标文件夹必须是同一主人
        this.pager.setFilterValue('userUuid', this.userUuid)



        this.pager.setFilterValue('dir', true)
        this.pager.httpFastPage()

      }
    },
    components: {
      NbExpanding
    },
    mounted() {
      this.refresh()
    }

  }
</script>

<style lang="less" rel="stylesheet/less">
  .folder-tree {
    .folder-block {
      padding: 5px 10px;
      line-height: 40px;
      &:hover {
        background: #eee;
      }
    }
    .ml23 {
      margin-left: 23px;
    }
  }

</style>
