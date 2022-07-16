<template>
  <div class="q-pa-md">
    <q-layout view="lHh lpr lFf" container style="height: 370px" class="shadow-2 rounded-borders">
      <q-header elevated>
        <q-toolbar>
          <q-avatar>
            <img src="https://cdn.quasar.dev/logo-v2/svg/logo-mono-white.svg">
          </q-avatar>

          <q-toolbar-title>
            Cadastro de clientes
          </q-toolbar-title>

          <q-btn flat round dense icon="whatshot" />
        </q-toolbar>
      </q-header>

      <q-page-container>
        <q-page padding id="corpo" size="20px">
            <q-item-label>Nome do cliente</q-item-label>
            {{edtCliente}}
            <q-input dense outlined ref="cliente" v-model="edtCliente" /><br />
            <q-item-label>Data de nascimento</q-item-label>
            <q-input dense outlined mask="##/##/####" class="nasc" ref="nascimento" type="text" v-model="edtDataNascimento"  /><br />
            <q-item-label>Endereço completo</q-item-label>
            <q-input  dense outlined ref="endereco" v-model="edtEndereco"  /><br />
            <div id="btn">
              <q-btn color="primary" label="Gravar" @click="postClientes()"/>
            </div>
        </q-page>
      </q-page-container>
    </q-layout>
      <div class="row" style="background-color: aqua;font-weight: bolder;">
        <div class="col">
          Código
        </div>
        <div class="col">
          Nome do cliente
        </div>
        <div class="col">
          Data de nascimento
        </div>
        <div class="col">
          Endereço
        </div>
        <div class="col" id="acao">
          Ação
        </div>
      </div>
      <div class="row" v-for="(lista,index) in rows" :key="index+1">
        <div class="col">
         {{lista.id}}
        </div>
        <div class="col">
         {{lista.nomeCliente}}
        </div>
        <div class="col">
          {{lista.nascimento}}
        </div>
        <div class="col">
          {{lista.endereco}}
        </div>
        <div class="col" id="icones">
          <q-icon 
            class="icone"
            name="delete"
            @mouseover="ativo = false"
            @click="deleteCLientes(lista.id)"
          />
          <q-icon 
            class="icone"
            name="update"
            @mouseover="ativo = false"
            @click="cliqueUpdatIcone(lista,index)"
          />
        </div>
      </div>
      <q-dialog v-model="alert">
      <q-card>
        <q-card-section>
          <div class="text-h6">Validação de campo</div>
        </q-card-section>

        <q-card-section class="q-pt-none">
          {{msgAlerta}}
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="OK" color="primary" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>
    <q-dialog v-model="updateTela">
      <q-card>
        <q-card-section>
          <div class="text-h6">Alteração de dados</div>
        </q-card-section>

            <q-item-label class="qInputModel">Nome do cliente</q-item-label>
            <q-input class="qInputModel" outlined ref="clienteModel" v-model="edtClienteModel" /><br />
            <q-item-label class="qInputModel">Data de nascimento</q-item-label>
            <q-input outlined mask="##/##/####" class="nascModel" ref="nascimentoModel" type="text" v-model="edtDataNascimentoModel"  /><br />
            <q-item-label class="qInputModel">Endereço completo</q-item-label>
            <q-input class="qInputModel" outlined ref="enderecoModel" v-model="edtEnderecoModel"  /><br />
            <div id="btn">
              <q-btn color="primary" label="Atualizar" v-close-popup @click="AtualizaModal()"/>
            </div>
            <br />
      </q-card>
    </q-dialog>
  </div>
  
</template>
<script>
import { ref } from 'vue'
import axios from "axios"
export default {
  
    mounted(){
        this.focusNome(); 
        this.getClientes();   
    },
    data(){
      return{
          selected: ref([]),
          edtCliente:'',
          edtDataNascimento:'',
          edtEndereco:'',
          rows : [],
          alert:false,
          msgAlerta:'',
          updateTela:false,
          edtClienteModel:'',
          edtDataNascimentoModel:'',
          edtEnderecoModel:'',
          indexGlobal:-1,
          idCliente:0

      }
    },
    methods:{
       incluirCliente(){
        if(this.validacaoCampos()){
          let objeto = {
                          Nome:this.edtCliente,
                          Nascimento:this.edtDataNascimento,
                          Endereco:this.edtEndereco,
                          Acao:''
                       }
          
          this.rows.push(objeto)
          this.statusCampos(false)
          this.$refs.cliente.focus()
         }
       },
       statusCampos(){
          this.edtCliente = ''
          this.edtDataNascimento = ''
          this.edtEndereco = ''
          this.focusNome(); 
       },
       focusNome(){
         this.$refs.cliente.focus()
       },
       validacaoCampos(){

          this.alert = false;

          if(this.edtCliente == ''){
            this.msgAlerta="Nome do cliente não informado."
            this.alert = true;
            this.$refs.cliente.focus()
            return false    
          }else if(this.edtDataNascimento ==''){
            this.msgAlerta="Data de nascimento do cliente não informado."
            this.alert = true;
            this.$refs.nascimento.focus()
            return false
          }else if(this.edtEndereco == ''){
             this.msgAlerta="Endereço do cliente não informado."
             this.alert = true;
             this.$refs.endereco.focus()
             return false
          }
          
          return true
       },
       cliqueIcone(index){

        this.rows.splice(index-1,1)

       },
       cliqueUpdatIcone(lista,index){

          this.updateTela = false;
          this.idCliente = lista.id
          this.edtClienteModel = lista.nomeCliente
          this.edtDataNascimentoModel = lista.nascimento
          this.edtEnderecoModel = lista.endereco
          this.indexGlobal = index
          this.updateTela = true;

       },
       AtualizaModal(){
         this.updateCliente(this.idCliente)
       },
       getClientes(){
        axios
        .get('https://api-cadastrocliente.herokuapp.com/clientes')
        .then(response => {
               this.rows = response.data
               console.log(response.data)
        })
        .catch(error => console.log(error))

       },
       postClientes(){

        let objCliente = {
                          nomeCliente:this.edtCliente,
                          nascimento:this.edtDataNascimento, 
                          endereco:this.edtEndereco
                        }
        axios
          .post('https://api-cadastrocliente.herokuapp.com/clientes',objCliente)
          .then(response => {
                this.rows = response.data
                console.log(response.data)
                this.getClientes()
                this.statusCampos()
          })
          .catch(error => console.log(error))
       },

       deleteCLientes(id){
        axios
          .delete('https://api-cadastrocliente.herokuapp.com/clientes/'+id)
          .then(response => {
                this.rows = response.data
                console.log(response.data)
                this.getClientes()
          })
          .catch(error => console.log(error))

       },
       updateCliente(idCliente){
        let objCliente = {
                          id:idCliente,
                          nomeCliente:this.edtClienteModel,
                          nascimento:this.edtDataNascimentoModel, 
                          endereco:this.edtEnderecoModel
                        }
        axios
          .post('https://api-cadastrocliente.herokuapp.com/clientes',objCliente)
          .then(response => {
                this.rows = response.data
                console.log(response.data)
                this.getClientes()
                this.statusCampos()
          })
          .catch(error => console.log(error))
       }

    },
    
    
}
</script>

<style lang="sass" scoped>
.row > div
  padding: 5px 10px
  background: rgba(86,61,124,.15)
  border: 1px solid rgba(86,61,124,.2)
.row + .row
  margin-top: 0,1rem
</style>

<style scoped>
.q-input{
  font-size: 14px;
  position: relative;
  width: 1000px;
}
.nasc{
  width: 200px;
}
#btn{
  text-align: center
}
.q-icon:hover{
  cursor: pointer;
}

.q-icon{
  font-size: 20px;
  position: relative;
}

#icones,#acao{
  text-align: center
}

.qInputModel{
  padding: 10px;
  width: 500px;
}

.nascModel{
  padding: 10px;
  width: 200px;
}

</style>

