<template>
  <div class="oschBox"> 
    <div class="logoBox clear">
        <img src="../assets/oschlogo.png" alt="" class="fl">
        <h1 class="logoTitle">Open Source Chain</h1>
    </div>
    <div class="walletBox"> 
        <div class="onelayer layer">
            <h1>OSCH Account Viewer</h1>           
            <p>可以用次钱包接受和发送osch到stellar网络</p>
        </div>
        <div class="twolayer layer">
            <h3>私钥:</h3>      
            <div class="input_box">
                <input type="password" v-model="secret" class="inputText" placeholder="请输入你的私钥"> 
            </div>
            <div class="input_box">
                <router-link :to="{ path: '/account/'+secret}" class="s-button">登入</router-link>
            </div>
        </div>
        <div class="threelayer layer">
            <h3>创建一个keypair:</h3>      
            <div class="input_box">
                <input type="button" value="生成"  class="s-button" @click="generkey">
            </div>
            <div class="keysBox" v-if="ifKeyBox">
                <h4>public key:</h4>    
                <p>{{newPublicKey}}</p>
                <h4>private key:</h4>
                <p>{{newPrivateKey}}</p>
            </div>
        </div>
        <div class="threelayer layer">
            <h3>使用上面创建的公钥领取初始osch(初始账号需要一笔osch激活，才可以正常使用)</h3>
            <input type="button" value="确认领取" class="s-button" @click="createAccount">
        </div>
    </div>
  </div>
</template>
<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      secret: "",
      ifKeyBox: false,
      newPublicKey: "",
      newPrivateKey: "",
      stellarServer: ""
    }
  },
  methods: {
    generkey: function(){
        var keypair = StellarSdk.Keypair.random();  
        var strkey = StellarSdk.StrKey;
        this.newPublicKey = strkey.encodeEd25519PublicKey(keypair._publicKey);
        this.newPrivateKey = strkey.encodeEd25519SecretSeed(keypair._secretSeed);
        this.ifKeyBox = true;
    },
    createAccount: function(){
        var _this = this;
        _this.stellarServer
            .loadAccount("GATTKBB6SHMYIX2BTKQDJ2P34RSTXBNH4IZYZYTTVX5CI72HZZHG5OWQ")
            .then(function(account){
                console.log(account);
                var transaction = new StellarSdk.TransactionBuilder(account)
                    .addOperation(StellarSdk.Operation.createAccount({
                      destination:_this.newPublicKey,
                      startingBalance: "20"  // in XLM
                    }))
                    .build();
                transaction.sign(StellarSdk.Keypair.fromSecret('SBEGIXOOTJ2HY6CJCP3QBBOUUZ32RBPOID4YLQZ5UVPLLHVQ7P2OVIO4'));
                _this.stellarServer.submitTransaction(transaction).then(function(res){
                    alert("账户初始化成功");
                })

            })
    }
  },
  mounted(){
    console.log(StellarSdk);
    //https://horizon-testnet.stellar.org
    //StellarSdk.setAllowHttp = true;
     //var config = new StellarSdk.Config();
    StellarSdk.Config.setAllowHttp(true);
    //console.log(config);
     //config._proto_.setAllowHttp(true);
    StellarSdk.Network.useTestNetwork();
    this.stellarServer = new StellarSdk.Server('https://horizon-testnet.stellar.org');
    //var server = new StellarSdk.Server('http://192.168.1.3:8000');
    // get a list of transactions that occurred in ledger 1400
    var publicKey = "GATTKBB6SHMYIX2BTKQDJ2P34RSTXBNH4IZYZYTTVX5CI72HZZHG5OWQ";
    var secretString = "SBEGIXOOTJ2HY6CJCP3QBBOUUZ32RBPOID4YLQZ5UVPLLHVQ7P2OVIO4";
    /*
    server
        .loadAccount(publicKey)
        .then(function(account){
            var transaction = new StellarSdk.TransactionBuilder(account)
                    // this operation funds the new account with XLM
                    .addOperation(StellarSdk.Operation.payment({
                        destination: "GCZYFPY5ZZ2ASKO6NS2ICBWAHMLD54BP5WUDWK7DWABQUWNDBA5ZGKKG",
                        asset: StellarSdk.Asset.native(),
                        amount: "2000"
                    }))
                    .build();
            transaction.sign(StellarSdk.Keypair.fromSecret(secretString)); // sign the transaction
            return server.submitTransaction(transaction);
        })
   */
    /*
    server.transactions()
        .forLedger(1400)
        .call().then(function(r){ console.log(r); });
    */ 
    // get a list of transactions submitted by a particular account
    /*
    server.transactions()
        .forAccount('GASOCNHNNLYFNMDJYQ3XFMI7BYHIOCFW3GJEOWRPEGK2TDPGTG2E5EDW')
        .call().then(function(r){ console.log(r); });
    */

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    .oschBox{ width: 1050px; margin: 0 auto; } 
    .logoBox{ text-align: left;}
    .logoBox img{ width: 100px;}
    .logoTitle{ color: #ed8d26; height: 100px; line-height: 100px;}
    .walletBox{ width: 720px; margin-left: 20px; margin-top: 25px; border: 1px solid #ed8d26; border-radius: 3px 3px 3px 3px; background: #f8d4ae;}
    .layer{ border-bottom: 1px solid #ed8d26;  padding: 30px 50px;}
    .onelayer p{ margin-top: 20px; } 
    .s-button {
        margin-top: 10px;
        display: inline-block;
        z-index: 1;
        padding: 0.35em 1.5em 0.3166em 1.5em;
        border: 1px solid #0691b7;
        box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.12), inset 0 -1px 0 rgba(0, 0, 0, 0.08);
        background: #08b5e5;
        color: white;
        text-decoration: none;
        border-radius: 3px;
        cursor: pointer;
    }
    .s-button:hover{
        background: #77b8d5;
        border-color: #0691b7; 
    }
    .inputText{
        margin-top: 5px;
        width: 100%;
        border: 1px solid #bdbfc0;
        color: #3a3f43;
        padding: 0.38em 1em 0.3466em 1em;
        border-radius: 3px; 
        height:30px; 
        line-height: 30px;
    }
    .keysBox{
        margin-top: 10px; 
    }
</style>
