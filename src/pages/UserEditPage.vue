<template>
  <van-form @submit="onSubmit">
    <van-field
        v-if="editUser.editKey !='gender' && editUser.editKey !='tags' && editUser.editKey !='avatar' && editUser.editKey !='profile'"
        v-model="editUser.currentValue"
        :name="editUser.editKey"
        :label="editUser.editName"
        :placeholder="`请输入${editUser.editName}`"
    />
    <van-cell-group>
      <van-field
          v-if="editUser.editKey =='profile'"
          v-model="editUser.currentValue"
          label="自我介绍"
          type="textarea"
          autosize
          rows="3"
          placeholder="请输入自我介绍"
      />
    </van-cell-group>
    <!--   修改性别 -->
    <van-radio-group style="padding: 10px; margin: 10px;" v-if="editUser.editKey =='gender'"
                     v-model="editUser.currentValue" direction="horizontal">
      <van-radio name="0">男</van-radio>
      <van-radio name="1">女</van-radio>
    </van-radio-group>

    <!--修改标签-->
    <div style="text-align: center">
      <van-button type="success" v-if="editUser.editKey =='tags'" size="small" @click="showTagList = true">选择你的标签</van-button>
      <van-row gutter="16" style="padding-top: 20px;padding-left: 70px;"
               v-if="editUser.editKey =='tags' && editUser.currentValue.length > 0">
        <van-col v-for="tag in activeIds " style="padding: 8px">
          <van-tag :show="true" closeable size="medium" type="primary" @close="doClose(tag)">
            {{ tag }}
          </van-tag>
        </van-col>
      </van-row>
    </div>
    <!--修改头像-->
    <van-field name="uploader"  v-if="editUser.editKey =='avatar'"  label="头像上传" >
      <template #input>
        <van-uploader v-model="avatarUrlList"
                      accept="image/*"
                      reupload max-count="1"
                      :after-read="uploadAvatar"/>
      </template>
    </van-field>
    <div style="margin: 16px;" v-if="editUser.editKey !='tags' && editUser.editKey !='avatar'">
      <van-button round block type="primary" native-type="submit">
        提交
      </van-button>
    </div>
  </van-form>
  <div style="margin: 16px;" v-if="editUser.editKey =='tags'">
    <van-button round block type="primary" native-type="submit" @click="updateTags">
      提交
    </van-button>
  </div>
  <div style="margin: 16px;" v-if="editUser.editKey =='avatar'">
    <van-button round block type="primary" native-type="submit" @click="updateAvatar">
      提交
    </van-button>
  </div>
</template>

<script setup lang="ts">
import {useRoute, useRouter} from "vue-router";
import { ref } from "vue";
import myAxios from "../plugins/myAxios";
import {Toast} from "vant";
import {getCurrentUser} from "../services/user";

const route = useRoute();
const router = useRouter();
const originTagList = [
  {
    text: '性别',
    children: [
      {text: '男', id: '男'},
      {text: '女', id: '女'},
    ],
  },
  {
    text: '年级',
    children: [
      {text: '高一', id: '高一',},
      {text: '高二', id: '高二',},
      {text: '高三', id: '高三',},
      {text: '大一', id: '大一',},
      {text: '大二', id: '大二'},
      {text: '大三', id: '大三'},
      {text: '大四', id: '大四'},
      {text: '研一', id: '研一'},
      {text: '研二', id: '研二'},
      {text: '研三', id: '研三'},
    ],
  },
  {
    text: '编程语言',
    children: [
      {text: 'Java', id: 'Java',},
      {text: 'Python', id: 'Python',},
      {text: 'C', id: 'C',},
      {text: 'Vue', id: 'Vue',},
      {text: 'React', id: 'React',},
      {text: 'Angular', id: 'Angular',},
      {text: 'C++', id: 'C++',},
      {text: 'Go', id: 'Go'},
      {text: 'C#', id: 'C#'},
    ],
  },
  {
    text: '目标',
    children: [
      {text: '考研', id: '考研',},
      {text: '春招', id: '春招',},
      {text: '秋招', id: '秋招',},
      {text: '考公', id: '考公',},
      {text: '竞赛', id: '竞赛',},
      {text: '转行', id: '转行',},
      {text: '跳槽', id: '跳槽',},
    ],
  },
  {
    text: '段位',
    children: [
      {text: '初级', id: '初级',},
      {text: '中级', id: '中级',},
      {text: '高级', id: '高级',},
      {text: '架构师', id: '架构师',},
    ],
  },
  {
    text: '状态',
    children: [
      {text: '乐观', id: '乐观',},
      {text: 'emo', id: 'emo',},
      {text: '一般', id: '一般',},
      {text: '单身', id: '单身',},
      {text: '已婚', id: '已婚',},
      {text: '有对象', id: '有对象',},
    ],
  },
]
const editUser = ref({
  editKey: route.query.editKey,
  currentValue: route.query.currentValue,
  editName: route.query.editName,
})

const onSubmit = async () => {
  const currentUser = await getCurrentUser();

  if (!currentUser) {
    Toast.fail('用户未登录');
    return;
  }

  console.log(currentUser, '当前用户')

  const res = await myAxios.post('/user/update', {
    'id': currentUser.id,
    [editUser.value.editKey as string]: editUser.value.currentValue,
  })
  console.log(res, '更新请求');
  if (res.code === 0 && res.data > 0) {
    Toast.success('修改成功');
    router.back();
  } else {
    Toast.fail('修改错误');
  }
};

</script>

<style scoped>

</style>
