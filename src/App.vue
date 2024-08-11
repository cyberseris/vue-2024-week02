<script setup>
  import { ref } from 'vue'
  import axios from 'axios'
  import Swal from 'sweetalert2'
  import validator from 'validator';

  const todoList = ref('')
  let account = ref('')
  let password = ref('')
  let nickname = ref('')
  let content = ref('')
  let isShow = ref(true)
  let isLogin = ref(false)
  let isEditable = ref(false)
  let token = ''

  const url = 'https://todolist-api.hexschool.io';
  
  const checkPasswordStrength = (password) => {
    let strength = 0;

    if(password.length >= 9){
      strength += 1;
    }

    if(password.match(/[a-z]/) && password.match(/[A-Z]/)){
      strength += 1;
    }

    if(password.match(/\d/)){
      strength += 1;
    }

    if(password.match(/[^a-zA-Z\d]/)){
      strength += 1;
    }

    return strength
  }
  
  const getCookie = (name) => {
      let pattern = new RegExp('(?:^|; )' + encodeURIComponent(name) + '=([^;]*)');
      let match = document.cookie.match(pattern);
      return match ? decodeURIComponent(match[1]) : null;
  }

  const deleteCookie = (name) => {
    document.cookie = name + '=; expires=Thu, 01 Jan 1970 00:00:00 GMT; path=/';
  }

  const signUp = async () => {
    if(account.value == password.value || account.value == nickname.value || password.value == nickname.value){
      Swal.fire({
        title: "註冊失敗",
        text: "帳號、密碼或暱稱不能相同",
        icon: "error"
      })
      .then(()=>{
        window.location.reload();
      })
    }

    if(validator.isEmail(account.value)){
      Swal.fire({
        title: "註冊失敗",
        text: "帳號格式不符",
        icon: "error"
      })
      .then(()=>{
        window.location.reload();
      })
    }

    if(checkPasswordStrength(password.value)<4){
      Swal.fire({
        title: "註冊失敗",
        text: "密碼至少包含數字、英文大小寫、特殊符號",
        icon: "error"
      })
      .then(()=>{
        window.location.reload();
      })
    }

    if(account.value && password.value && nickname.value){
        await axios.post(`${url}/users/sign_up`,{
          email: account.value,
          password: password.value,
          nickname: nickname.value
        })
        .then((res) => {
          if(res.data.status){
            Swal.fire({
              title: "註冊成功",
              text: "恭喜您註冊成功",
              icon: "success"
            })
            window.location.reload();  
          }else{
            Swal.fire({
              title: "註冊失敗",
              text: "註冊失敗",
              icon: "error"
            })
            window.location.reload();
          }
        })
        .catch((error) => {
          console.log("Error: ", error)
        })
    }else{
      Swal.fire({
        title: "輸入錯誤",
        text: "資料不得為空",
        icon: "warning"
      })
      window.location.reload();          
    }
  }

  const signIn = async () => {
    if(account.value && password.value){
      await axios.post(`${url}/users/sign_in`, {
        email: account.value,
        password: password.value
      })
      .then(res => {
        console.log("signIn", res.data)
        if(res.data.status){
          let date = new Date();
          date.setTime(date.getTime()+60*60*1*1000);
          date = date.toUTCString();
          document.cookie = `hexschoolTodo=${res.data.token}`+"expires="+date+";";
          Swal.fire({
            title: "登入成功",
            icon: "success"
          }).then((result) => {
            window.location.reload();
          }); 
        }else{
          Swal.fire({
            title: "登入失敗",
            text: "請重新登入",
            icon: "error"
          }).then((result) => {
            window.location.reload();
          }); 
        }
      })
      .catch(error => {
        console.log("Error: ", error)
      })
    }else{
      Swal.fire({
        title: "輸入錯誤",
        text: "資料不得為空",
        icon: "warning"
      }).then((result) => {
        window.location.reload();
      });
    }
  }

  const checkout = async () => {
    await axios.get(`${url}/users/checkout`, {
            headers: {
              Authorization: token
            }
          })
          .then(res => {
            alert(`${res.data.nickname} 您的資料是正確的`)
          })
          .catch(error => {
            console.log("Error: ", error)
          })
  }

  const signOut = async () => {
    await axios.post(`${url}/users/sign_out`, {}, {
      headers: {
              Authorization: token
      }
    })
    .then(res => {
      if(res.data.status){
        Swal.fire({
            title: "登出成功",
            icon: "success"
          }).then((result) => {
            window.location.reload();
          }); 
      }

    deleteCookie('hexschoolTodo')
    })
    .catch(err => {
      console.log("Error: ", err)
    })
  }

  const getTodos = async () => {
    token = getCookie('hexschoolTodo')?.split('expires')[0]
    if(token){
      await axios.get(`${url}/todos`, {
        headers:{
          Authorization:token
        }
      })
      .then(res => {
        todoList.value = res.data.data  
        todoList.value.forEach(todo => {
          todo.isEditable = false
        })
      })
      .catch(err => {
        console.log("Error: ", err)
      })
    }
  }
  
  const addTodo = async () => {
    await axios.post(`${url}/todos`, {content: content.value} , {
      headers:{
        Authorization:token
      }
    })
    .then(res => {
      if(res.data.status){
        Swal.fire({
          title: "新增成功",
          text: "成功新增待辦事項",
          icon: "success"
        }).then((result) => {
          window.location.reload();
        }); 
      }else{
        Swal.fire({
          title: "新增失敗",
          text: "新增待辦事項失敗",
          icon: "error"
        }).then((result) => {
          window.location.reload();
        }); 
      }
    })
    .catch(error => {
      console.log("Error: ", error)
    })
  }

  const updateTodo = async (todo) => {
    console.log("todo.id", todo.id)
    await axios.put(`${url}/todos/${todo.id}`, {content:todo.content}, {
      headers: {
        Authorization: token
      }
    })
    .then(res => {
      if(res.data.status){
        Swal.fire({
          title: "更新成功",
          text: "成功更新待辦事項",
          icon: "success"
        }).then((result) => {
          window.location.reload();
        }); 
      }
    })
    .catch(err => {
      console.log("Error:　", err)
    })
  }

  const deleteTodo = async (id) => {
    Swal.fire({
      title: "確定刪除?",
      text: "你確定要刪除這個待辦事項？",
      icon: "warning",
      showCancelButton: true,
      confirmButtonColor: "#3085d6",
      cancelButtonColor: "#d33",
      confirmButtonText: "刪除",
      cancelButtonText: "取消"
    }).then((result) => {
      if(result.isConfirmed){
        axios.delete(`${url}/todos/${id}`,{
            headers:{
              Authorization:token
            }
          })
          .then(res => {
            if(res.data.status){
              Swal.fire({
                title: "刪除成功",
                text: "成功刪除待辦事項",
                icon: "success"
              }).then((result) => {
                window.location.reload();
              }); 
            }
          })
          .catch(err => {
            console.log("Error: ", err)
          })
      }
    });
  }
  
  onMounted:{
    getCookie('hexschoolTodo')?.split('expires')[0]?isLogin.value=true:isLogin.value=false;
    getTodos();
  }
</script>

<template>
  <div>
    <form class="row g-3 mx-auto d-flex flex-column w-320 border border-radius gradient" v-if="isShow && !isLogin">
      <h1 class="mb-2 fs-3">todo API</h1>
      <div class="col-md-12 d-flex justify-content-center align-items-center">
        <label for="validationCustom01" class="form-label">帳號：</label>
        <div class="d-flex flex-column">
          <input type="text" class="form-control" id="validationCustom01" placeholder="ex:test@test.com" v-model="account">
        </div>
      </div>
      <div class="col-md-12 d-flex justify-content-center align-items-center">
        <label for="validationCustom02" class="form-label">密碼：</label>
        <div class="d-flex flex-column">
          <input type="text" class="form-control" id="validationCustom02" placeholder="********" v-model="password">
        </div>
      </div>

      <div class="col-md-12 d-flex justify-content-center mb-5">
        <button class="btn btn-primary btn-sm me-3"  @click.prevent="signIn">登入</button>
        <button class="btn btn-success btn-sm"  @click.prevent="isShow=!isShow">註冊</button>
      </div>
    </form>

    <form class="row g-3 mx-auto d-flex flex-column w-320 border border-radius gradient" v-if="!isShow && !isLogin">
      <h1 class="mb-2 fs-3">todo API</h1>
      <div class="col-md-12 d-flex justify-content-center align-items-center">
        <label for="validationCustom01" class="form-label">帳號：</label>
        <div class="d-flex flex-column">
          <input type="text" class="form-control" id="validationCustom01" placeholder="ex:test@test.com" v-model="account">
        </div>
      </div>
      <div class="col-md-12 d-flex justify-content-center align-items-center">
        <label for="validationCustom02" class="form-label">密碼：</label>
        <div class="d-flex flex-column">
          <input type="password" class="form-control" id="validationCustom02" placeholder="********" v-model="password">
        </div>
      </div>
      <div class="col-md-12 d-flex justify-content-center align-items-center">
        <label for="validationCustom02" class="form-label">暱稱：</label>
        <div class="d-flex flex-column">
          <input type="text" class="form-control" id="validationCustom02" placeholder="hex" v-model="nickname">
        </div>
      </div>
      <div class="col-md-12 d-flex justify-content-center mb-5">
        <button class="btn btn-success btn-sm me-3"  @click.prevent="signUp">註冊</button>
        <button class="btn btn-primary btn-sm"  @click.prevent="isShow=!isShow">回到登入頁面</button>
      </div>
    </form>

    <form class="row g-3 mx-auto d-flex flex-column align-items-center w-320 border border-radius gradient" v-if="isShow && isLogin">
      <h1 class="mb-2 fs-3">待辦事項</h1>
      <div class="col-md-12 d-flex justify-content-center align-items-center mb-3">
        <div class="d-flex">
          <input type="text" class="form-control w-210" id="validationCustom01" placeholder="請輸入待辦事項" v-model="content">
          <button class="btn btn-secondary btn-sm me-3"  @click.prevent="addTodo">新增待辦</button>
        </div>
      </div>
      
      <div class="border rounded bg-white pt-2 pb-2" style="width:300px" v-if="!todoList.length">尚無待辦事項 {{  }} </div>
      <div class="border rounded bg-white pt-2" style="width:300px" v-else>  
        <tr class="d-flex justify-content-between mb-2" v-for="todo in todoList" :key="todo.id" style="width:300px">
          <span class="me-auto" v-if=(!todo.isEditable)>{{ todo.content }}</span>
          <input type="text" v-model="todo.content" v-if=(todo.isEditable)>
          <span v-if=(!isEditable)><button class="btn btn-outline-primary btn-sm me-1"  @click.prevent="todo.isEditable = !todo.isEditable; isEditable = !isEditable">編輯</button>
            <button type="button" class="btn btn-outline-danger btn-sm me-3"  @click.prevent="deleteTodo(todo.id)">刪除</button>
          </span>
          <span v-else>
            <button class="btn btn-outline-secondary btn-sm me-3"  @click.prevent="todo.isEditable = !todo.isEditable;isEditable = !isEditable;updateTodo(todo);">確認</button>
          </span>  
        </tr>
      </div>


      <div class="col-md-12 d-flex justify-content-center mb-4">
        <button class="btn btn-primary btn-sm me-3"  @click.prevent="signOut">登出</button>
        <button class="btn btn-success btn-sm"  @click.prevent="checkout">驗證 token</button>
      </div>
    </form>
  </div>
</template>

<style scoped>
.fz-10{
  font-size: 10px;
}

.w-210{
  width: 210px;
}

.w-320{
  width: 320px;
}

.border-radius{
  border-radius: 20px;
}
</style>
