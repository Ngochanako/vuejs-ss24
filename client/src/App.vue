
<template>
   <v-container grid-list-xs>
    <!-- Loading -->
    <div v-show="loader" class="wrap-loader">
      <div class="loader"></div>
    </div>
    <!-- Modal add new post -->
     <div v-if="modalAdd" class="modal">
         <v-card class="formModal" width="500">
             <v-card-title primary-title style="display: flex; justify-content: space-between;">
              <div style="font-weight: 600;">Thêm mới bài viết</div>
              <v-icon @click="closeModalAdd">mdi-close</v-icon>
             </v-card-title>
             <v-form ref="form" @submit.prevent="addPost">
                <v-card-text>
                  <div>Tên bài viết</div>
                  <v-text-field
                    name="name"
                    id="id"
                    density="compact"
                    variant="solo"
                    :rules="nameRules"
                    required
                    v-model="post.title"
                  ></v-text-field>
                  <div>Hình ảnh</div>
                  <v-text-field
                    name="name"
                    id="id"
                    density="compact"
                    variant="solo"
                    :rules="imgRules"
                    required
                    v-model="post.img"
                  ></v-text-field>
                  <div>Ngày tạo bài viết</div>
                  <v-text-field
                    name="name"
                    id="id"
                    density="compact"
                    variant="solo"
                    type="date"
                    :rules="dateRules"
                    required
                    v-model="post.created_at"
                  ></v-text-field>
                </v-card-text>
                <VCardActions>
                  <v-btn color="success" block="" type="submit">Đồng ý</v-btn>
                </VCardActions>
             </v-form>
         </v-card>
     </div>
      <header style="display: flex;justify-content: space-between;">
         <div style="display: flex; gap: 20px;">
            <v-text-field
              name="name"
              label="Tìm kiếm"
              id="id"
              variant="outlined"
              density="compact"
              width="200"
              v-model="searchQuery"
              @input="onInput"
            ></v-text-field>
            <v-select
              :items="['Đã xuất bản','Ngừng hoạt động']"
              v-model="valueSelect"
              label="Lọc bài viết"
              density="compact"
              width="200"
              variant="solo"
              @update:modelValue="changeSelect"
            ></v-select>
         </div>
         <v-btn @click="openModalAdd" color="success">Thêm mới bài viết </v-btn>
      </header>
      <main>
          <v-table>
            <thead>
              <tr>
                <th>STT</th>
                <th>Tiêu đề</th>
                <th>Ngày đăng</th>
                <th>Hình ảnh</th>
                <th>Trạng thái</th>
                <th>Chức năng</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, index) in posts" :key="item.id">
                  <td>{{ index + 1 }}</td>
                  <td>{{ item.title }}</td>
                  <td>{{ item.created_at.split('-').reverse().join('-') }}</td>
                  <td style="padding: 10px;"><img :src="item.img" alt="image" width="100" height="100"></td>
                  <td>
                    <v-chip v-if="item.status" color="primary">Đã xuất bản</v-chip>
                    <v-chip v-else color="red">Ngừng hoạt động</v-chip>
                  </td>
                  <td>
                     <v-row style="gap: 10px;">
                        <v-btn color="orange" @click="editItem(item)">Sửa</v-btn>
                        <v-btn color="error" @click="deleteItem(item.id)">Xóa</v-btn>          
                     </v-row>
                  </td>
               </tr>
            </tbody>
          </v-table>
      </main>
   </v-container>
</template>

<script setup>
import axios from 'axios';
import debounce from 'lodash.debounce';
import { onMounted, reactive, ref } from 'vue';
import { VCardActions } from 'vuetify/lib/components/index.mjs';
const valueSelect=ref('')
const posts=reactive([]);
const loader=ref(false);
const modalAdd=ref(false);
const form=ref();
const searchQuery=ref('');
const post=reactive({
  id: '',
  title: '',
  img: '',
  created_at: '',
  status: false
})
const typeSubmit=ref('add');
const nameRules=[
  v=>!!v||'Tên bài viết không được để trống',
  v=>!(posts.find(p=>p.title==v))||'Tên bài viết không được trùng'
];
const imgRules=[
  v=>!!v||'Hình ảnh không được để trống'
];
const dateRules=[
  v=>!!v||'Ngày tạo bài viết không được để trống'
];
const getPostsAPI=()=>{
  posts.length=0;
  axios.get('http://localhost:3000/posts')
  .then(res=>{
    posts.push(...res.data)
  })
  .catch(err=>console.log(err));
}
onMounted(()=>{
  getPostsAPI(); 
})
const resetPost=()=>{
  post.id='';
  post.title='';
  post.img='';
  post.created_at='';
  post.status=false;
}
//add new post
const addPost=async()=>{
    //validate form
    const {valid}=await form.value.validate();
    if(!valid){
      return;
    }
    loader.value=true;
    if(typeSubmit.value=='add'){
       post.id=Date.now();
       posts.push({...post});
       await axios.post('http://localhost:3000/posts', post);
    }else{
       const postIndex=posts.findIndex(p=>p.id==post.id);
       posts[postIndex]={...post};
       await axios.put(`http://localhost:3000/posts/${post.id}`, post);
       typeSubmit.value='add';
    } 
    modalAdd.value=false;  
    setInterval(()=>{
      loader.value=false;
    },1000)
    resetPost();
}
//close modal add
const closeModalAdd=()=>{
  modalAdd.value=false;
  resetPost();
}
//open modal add
const openModalAdd=()=>{
  modalAdd.value=true;
}
//edit Post
const editItem=(item)=>{
  post.id=item.id;
  post.title=item.title;
  post.img=item.img;
  post.created_at=item.created_at;
  post.status=item.status;
  typeSubmit.value='edit';
  modalAdd.value=true;
}
//delete Post
const deleteItem=(id)=>{
  const confirmDelete=window.confirm('Bạn có chắc muốn xóa bài viết này?');
  if(confirmDelete){
    loader.value=true;
    axios.delete(`http://localhost:3000/posts/${id}`)
    .then(()=>{
       const postIndex=posts.findIndex(p=>p.id==id);
       posts.splice(postIndex,1);
     })
    .catch(err=>console.log(err));
     setInterval(()=>{
      loader.value=false;
    },1000)
  }
}
const changeSelect=()=>{
  
  const value=valueSelect.value;
  if(value=='Đã xuất bản'){
    posts.length=0;
    axios.get('http://localhost:3000/posts?status=true')
  .then(res=>{
    posts.push(...res.data)
  })
  .catch(err=>console.log(err));
  }else if(value=='Ngừng hoạt động'){
    posts.length=0;
    axios.get('http://localhost:3000/posts?status=false')
  .then(res=>{
    posts.push(...res.data)
  })
  .catch(err=>console.log(err));
  }
}
//search value
// Hàm gọi API tìm kiếm
const searchAPI = (query) => {
  axios.get(`http://localhost:3000/posts?title_like=${query}`)
    .then(response => {
       posts.length = 0;
       posts.push(...response.data);
    })
    .catch(error => {
      console.error(error);
    });
};

// Sử dụng debounce để giới hạn tần suất gọi API
const debouncedSearch = debounce((query) => {
  searchAPI(query);
}, 500);  // Đợi 500ms sau khi người dùng dừng gõ

// Hàm được gọi mỗi khi người dùng gõ từ khóa tìm kiếm
const onInput = () => {
  debouncedSearch(searchQuery.value);
};
</script>

<style>
/* HTML: <div class="loader"></div> */
.wrap-loader {
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background-color: rgba(125, 121, 121, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 100;
}
/* HTML: <div class="loader"></div> */
.loader {
 
  margin: auto;
  width: 50px;
  aspect-ratio: 1;
  display:grid;
  -webkit-mask: conic-gradient(from 15deg,#0000,#000);
  animation: l26 1s infinite steps(12);
}
.loader,
.loader:before,
.loader:after{
  background:
    radial-gradient(closest-side at 50% 12.5%,
     #fbfafa 96%,#0000) 50% 0/20% 80% repeat-y,
    radial-gradient(closest-side at 12.5% 50%,
     #f7f6f6 96%,#0000) 0 50%/80% 20% repeat-x;
}
.loader:before,
.loader:after {
  content: "";
  grid-area: 1/1;
  transform: rotate(30deg);
}
.loader:after {
  transform: rotate(60deg);
}

@keyframes l26 {
  100% {transform:rotate(1turn)}
}
/* modal */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0,0,0,0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 100;
}
.formModal{
  margin: auto;
}
</style>