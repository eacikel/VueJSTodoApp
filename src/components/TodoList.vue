<template>

    <div>
        <input type="text" class="todo-input" placeholder="Kendine biraz iş yüklemeye ne dersin :)" v-model="newTodo" @keyup.enter="addTodo"> <!-- enter tuşundan elini çekince addTodo methodu çağrılıyor -->
        <transition-group name="fade" enter-active-class="animated fadeInUp" leave-active-class="animated fadeOutDown">
        <div v-for="(todo, index) in todosFiltered" :key="todo.id" class="todo-item"> <!-- todos array içindeki her eleman for döngüsü ile todo olarak alınıyor bu işlem için key olarak todo2nun id'si kullanılıyor -->
            <div class="todo-item-left">
                <input type="checkbox" v-model="todo.completed">
                <div v-if="!todo.editing" @dblclick="editTodo(todo)" class="todo-item-label" :class=" { completed : todo.completed } ">{{todo.title}}</div> <!-- editing durumu false ise liste elemanı gibi listeniyor ve inputa çift tıklanınca editTodo methodu çağrılıyor bu methoda todo parametre olarak gönderiliyor methodda editing true yapılıyor--> <!-- class daha önce kullandığımız için :class ile bind ediyoruz. completed : todo.completed ise eğer todo.completed true ise classı completed yap demektir (yani if) -->
                <input v-else type="text" class="todo-item-edit" v-model="todo.title" @blur="doneEdit(todo)" @keyup.enter="doneEdit(todo)" @keyup.esc="cancelEdit(todo)" v-focus> <!-- editing durumu true olduğu anlarda burası çalışıyor ve düzenlenebilir inputa dönüşmüş oluyor değişiklikten sonra herhangi bi yere ya da enter'a tıklarsak doneEdit methodu çağırılıyor v-focus ile dclick yapılınca cursorun sona yerleşmesi sağlanıyor esc'ye tıklandığında canceledit methodu çağrılıyor-->
            </div>
            <div class="remove-item" @click="removeTodo(index)"> <!-- x işaretine tıklayınca removeTodo methodu çağrılacak ve bu method o todo'nun index'ini parametre olarak alacak -->
                &times;
            </div>
        </div>
        </transition-group>
        <div class="extra-container">
            <div><label><input type="checkbox" :checked="!anyRemaining" @change="checkAllTodos">Hepsini Seç</label></div><!-- :checked="!anyRemaining" bu bind işlemi ile tüm todo'lar seçildiğinde check all box'ını aktif, birisi iptal edildiğinde ise deaktif ediyoruz -->
            <div>{{ remaining }} items left</div>
        </div>
        <div class="extra-container"> <!-- All Active Completed butonlarının yer aldığı alan -->
            <div>
                <button :class="{ active: filter == 'all' }" @click="filter = 'all'">Hepsi</button> <!-- @click="filter = 'all' ile bu butona tıklanırsa filter=all yap --> <!-- { active: filter == 'all' } ile filter=all ise class'ı active yap -->
                <button :class="{ active: filter == 'active' }" @click="filter = 'active'">Aktifler</button> <!-- @click="filter = 'active' ile bu butona tıklanırsa filter=active yap --> <!-- { active: filter == 'active' } ile filter=active ise class'ı active yap -->
                <button :class="{ active: filter == 'completed' }" @click="filter = 'completed'">Tamamlananlar</button> <!-- @click="filter = 'completed' ile bu butona tıklanırsa filter=completed yap --> <!-- { active: filter == 'completed' } ile filter=completed ise class'ı active yap -->
            </div>
            <div>
                <transition name="fade">
                    <button v-if="showClearCompletedButton" @click="clearCompleted">Clear Completed</button>
                </transition>
            </div>
        </div>
    </div>
</template>

<script>

export default {
  name: 'todo-list',
  data () {
    return {
      newTodo: '', // oluşan her todo'yu atacağımız obje
      idForTodo: 1, // değeri 3 olan bir değişken
      beforeEditCache: '', //todo'ların değişiklik öncesi son halini tutacak olan başlangıçta boş bir string değişken
      filter: 'all', // başlangıçta tüm todo'lar görünsün diye filter değişkenine default olarak all atadık
      todos: []
    }
  },
  computed: {
      remaining() {
          return this.todos.filter(todo => !todo.completed).length //tamamlanmamış todo'ların sayısını verir
      },
      anyRemaining() { //tüm todo'lar seçildiğinde check all box'ını aktif, birisi iptal edildiğinde ise deaktif ediyoruz
          return this.remaining != 0
      },
      todosFiltered(){
          if(this.filter == 'all'){
              return this.todos // filter değişkeni all ise tüm todo'ları göster
          }
          else if(this.filter == 'active'){
            return this.todos.filter(todo => !todo.completed) // filter değişkeni active ise completed olmayan todo'ları göster
          }
          else {
            return this.todos.filter(todo => todo.completed) // filter değişkeni active ise completed olan todo'ları göster
          }

          return this.todos // default durum yani filter==all ile aynı durum
      },
      showClearCompletedButton() {
            return this.todos.filter(todo => todo.completed).length > 0 // en az 1 tane completed true olan todo varsa çalışır
      }
  },
  directives: {
      focus: {
          inserted: function (el) {
              el.focus()
          }
      }
  },
  methods: {
      addTodo(){

          if(this.newTodo.trim().length == 0 ){ // input boşken entera basıldığında ekleme yapmasın id gereksiz artmasın diye kontrol
              return
          }

          this.todos.push({ // todo ekleme işlemi gerçekleşiyor
              id: this.idForTodo, // id'sine idForTodo değişkenindeki değer atılıyor
              title: this.newTodo, // title olarak inputtaki değer atılıyor
              completed: false // todo yeni eklendiği için completed durumu false atılıyor
          })

          this.newTodo = '' // newTodo objesi boşaltılıyor
          this.idForTodo++ // idForTodo değişkeni bir sonraki todo için 1 arttırılıyor
      },
      editTodo(todo) {
          this.beforeEditCache = todo.title // editlenen todo'nun edit öncesi son title'ını tutuyoruz
          todo.editing = true
      },
      doneEdit(todo) {
          if(todo.title.trim() == '' ){ // editi tamamlarken todo'nun başlığı boş mu diye baktığımız kontrol
              todo.title = this.beforeEditCache // eğer başlık silinmiş ve edit gerçekleştirilmek isteniyorsa izin vermeyip bağlığın yerine son halini koyuyoruz
          }
          todo.editing = false // edit işlemi tamamlanınca durumu false yapıyoruz ki ön yüzde else durumuna girmeyip kullanıcıya input değilde liste elemanı gibi görünsün diye
      },
      cancelEdit(todo) {
          todo.title = this.beforeEditCache // esc'ye basıldığında çalıştığı için edit yapılan todo'nun cache'deki son halini alıyor yani değişiklikleri iptal etmiş oluyor
          todo.editing = false
      },
      removeTodo(index) {
          this.todos.splice(index, 1) // todos listesinden gelen index'e sahip olan todo'yu ayır
      },
      checkAllTodos() { // checked all box'ı aktifken box'a tıklanıp deaktif edildiğinde seçili tüm checkbox'ları iptal etmesi
          this.todos.forEach((todo) => todo.completed = event.target.checked)
      },
      clearCompleted() {
          this.todos = this.todos.filter(todo => !todo.completed) // clear completed butonuna basınca çalışan bu method completed true olan durumu false yaparak silmiş oluyor
      }
  }
}
</script>

<style>
    @import url("https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.5.2/animate.min.css");

    .todo-input { /* input için css */
        width: 100%;
        padding: 10px 18px;
        font-size: 18px;
        margin-bottom: 16px;
    }

    .todo-input:focus {
        outline: 0;
    }
    
    .todo-item { /* listedeki todo'lar için css */
        margin-bottom: 12px;
        display: flex;
        align-items: center;
        justify-content: space-between;
    }

    .remove-item { /* silme iconu olan x için css */
        cursor: pointer;
        margin-left: 14px;
    }

    .remove-item:hover {
        color: black;
    }

    .todo-item-left {
        display: flex;
        align-items: center;
    }

    .todo-item-label {
        padding: 10px;
        border: 1px solid white;
        margin-left: 12px;
    }

    .todo-item-edit {
        font-size: 24px;
        color: #2c3e50;
        margin-left: 12px;
        width: 100%;
        padding: 10px;
        border: 1px solid #ccc;
        font-family: 'Avenir', Helvetica, Arial, sans-serif;
    }

    .todo-item-edit:focus {
        outline: none;
    }

    .completed { /* checkbox işaretlenen todo'ların css'i */
        text-decoration: line-through;
        color: red;
    }

    .extra-container {
        display: flex;
        align-items: center;
        justify-content: space-between;
        font-size: 16px;
        border-top: 1px solid lightgreen;
        padding-top: 14px;
        margin-bottom: 14px;
    }

    button {
        font-size: 14px;
        background-color: white;
        appearance: none;
    }

    button:hover {
        background-color: lightgreen;
    }

    button:focus {
        outline: none;
    }

    .active {
        background: lightgreen;
    }

    /* animasyon için css */
    .fade-enter-active, .fade-leave-active {
        transition: opacity .2s;
    }
    
    .fade-enter, .fade-leave-to {
        opacity: 0;
    }
</style>
