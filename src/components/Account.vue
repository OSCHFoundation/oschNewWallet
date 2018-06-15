<template>
  <div class="oschBox"> 
    <div class="logoBox clear">
        <img src="../assets/oschlogo.png" alt="" class="fl">
        <h1 class="logoTitle fl">Open Source Chain</h1>
        <router-link :to="{ path: '/'}">登出</router-link>
    </div>
    <div class="money_box">
        <div class="money_box1">
            <h3>你的余额:</h3>
            <p><span>{{oschNum}}</span>&nbspOSCH</p>
            <h3 class="title1">你的公钥:</h3>
            <p>{{publicKey}}</p>
        </div>
     </div>
    <div class="sendTran">
        <div class="sendTran1">
            <h2>发送osch</h2>
            <div class="inputBox">
               <label for="toPublic">to:</label>
            </div>
            <div class="inputBox">
                <input class="inputText" type="text" id="toPublic" v-model="toPublic" placeholder="请输入公钥">
            </div>
            <div class="inputBox">
                <label for="sendNum">数量:</label>
            </div>
            <div class="inputBox">
                <input type="text" class="inputText" id="sendNum" v-model="toOschNum" placeholder="输入发送数量">
            </div>
            <div class="inputBox">
                <input type="button" class="s-button" value="发送OSCH" @click="sendClick">
            </div>
        </div>
    </div>
   <div class="historyBox">
        <div class="historyBox1">
            <h2>Transaction History</h2>
             <el-table
                :data="tableData"
                stripe
                style="width: 100%">
                <el-table-column
                  prop="time"
                  label="Create_Time"
                  width="200">
                </el-table-column>
                <el-table-column
                  prop="num"
                  label="Amount"
                  width="200">
                </el-table-column>
                 <el-table-column
                  prop="to"
                  label="To"
                  width="230">
                </el-table-column>
                <el-table-column
                  prop="operation"
                  label="Operation ID">
                </el-table-column>
            </el-table>
        </div>
   </div>
  </div>
</template>
<script>
export default {
  name: 'HelloWorld',
  data () {
    return {
      sercet: this.$route.params.id,
      publicKey: "13212",
      oschNum: "0",
      toPublic: "",
      toOschNum: "",
      account: "",
      tableData: []
    }
  },
  methods: {
    sendClick: function(){
        var _this = this;
        var server = new StellarSdk.Server('https://horizon-testnet.stellar.org');
        var transaction = new StellarSdk.TransactionBuilder(_this.account)
            // this operation funds the new account with XLM
            .addOperation(StellarSdk.Operation.payment({
                destination: _this.toPublic,
                asset: StellarSdk.Asset.native(),
                amount: _this.toOschNum 
            }))
        .build();
        transaction.sign(StellarSdk.Keypair.fromSecret(this.sercet)); // sign the transaction
        server.submitTransaction(transaction).then(function(res){
            alert("发送成功");
            location.reload() 
        });
    } 
  },
  mounted(){
    var _this = this;
    StellarSdk.Config.setAllowHttp(true);
    StellarSdk.Network.useTestNetwork();
    var server = new StellarSdk.Server('https://horizon-testnet.stellar.org');
    var secretString = "SBEGIXOOTJ2HY6CJCP3QBBOUUZ32RBPOID4YLQZ5UVPLLHVQ7P2OVIO4";
    var strkey = StellarSdk.StrKey;
    var arrPrivate = strkey.decodeEd25519SecretSeed(this.sercet);
    var keypair = new StellarSdk.Keypair({
        type: "ed25519",
        secretKey: arrPrivate 
    });
    this.publicKey = strkey.encodeEd25519PublicKey(keypair.rawPublicKey());

    //获取历史交易
    server.transactions()
        .forAccount(this.publicKey)
        .call()
        .then(function (page) {
            for(var i=0; i<page.records.length; i++){
                page.records[i].operations().then(function(res){
                    if(res.records[0].from == _this.publicKey){
                        var ob = {
                            time: res.records[0].created_at,
                            operation: res.records[0].transaction_hash, 
                            num: "-"+res.records[0].amount,
                            to: res.records[0].to
                        };            
                        _this.tableData.push(ob);
                    }else if(res.records[0].type=="create_account"){
                         var ob = {
                            time: res.records[0].created_at,
                            operation: res.records[0].transaction_hash, 
                            num: "+"+res.records[0].starting_balance,
                            to: res.records[0].to
                        };            
                        _this.tableData.push(ob);
                    }else{
                        var ob = {
                            time: res.records[0].created_at,
                            operation: res.records[0].transaction_hash, 
                            num: "+"+res.records[0].amount,
                            to: res.records[0].to
                        };            
                        _this.tableData.push(ob);
                    } 
                })
            }
            return page.next();
        })
        .then(function (page) {
            for(var i=0; i<page.records.length; i++){
                page.records[i].operations().then(function(res){
                    if(res.records[0].from == _this.publicKey){
                        var ob = {
                            time: res.records[0].created_at,
                            operation: res.records[0].transaction_hash, 
                            num: "-"+res.records[0].amount,
                            to: res.records[0].to
                        };            
                        _this.tableData.push(ob);
                    }else{
                        var ob = {
                            time: res.records[0].created_at,
                            operation: res.records[0].transaction_hash, 
                            num: "+"+res.records[0].amount,
                            to: res.records[0].to

                        };            
                        _this.tableData.push(ob);
                    } 
                })
            }
            return page.next();
        })
        .catch(function (err) {
            console.log(err);
        });     
    //找到账户信息
    server
        .loadAccount(this.publicKey)
        .then(function(account){
            _this.account = account;
            _this.oschNum = account.balances[0].balance;
        })

    //console.log(keypair.rawPublicKey());
   
    //("GATTKBB6SHMYIX2BTKQDJ2P34RSTXBNH4IZYZYTTVX5CI72HZZHG5OWQ")
   // var keypair = StellarSdk.Keypair.random();  
     
   // kepair.fromRawEd25519Seed("SBEGIXOOTJ2HY6CJCP3QBBOUUZ32RBPOID4YLQZ5UVPLLHVQ7P2OVIO4");
    
    //console.log(keypair)
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
    .oschBox{ } 
    .logoBox{  width: 1050px; margin: 0 auto;text-align: right; line-height: 100px; padding-right: 100px;}
    .logoBox a{ text-decoration:underline; font-size: 18px;}
    .logoBox img{ width: 100px;}
    .logoTitle{ color: #ed8d26; height: 100px; line-height: 100px;}
    .walletBox{ width: 720px; margin-left: 20px; margin-top: 25px; border: 1px solid #ed8d26; border-radius: 3px 3px 3px 3px; background: #f8d4ae;}
    .layer{ border-bottom: 1px solid #ed8d26;  padding: 30px 50px;}
    .onelayer p{ margin-top: 30px; } 
    .s-button {
        z-index: 1;
        padding: 10px 35px;
        margin-top: 5px;
        border: 1px solid #0691b7;
        box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.12), inset 0 -1px 0 rgba(0, 0, 0, 0.08);
        background: #08b5e5;
        color: white;
        text-decoration: none;
        border-radius: 3px;
        cursor: pointer;
    }
    .money_box{
        padding-top: 2em;
        padding-bottom: 2em;
        border-top: 1px solid #dedfe0;
        border-bottom: 1px solid #dedfe0;
        background: #f4f4f5;
    }
    .money_box1{
        width: 1050px; 
        margin: 0 auto;
        padding-left: 60px; 
    }
    .money_box1 h3{ font-size: 16px;  word-wrap: break-word;  word-break: break-word;}
    .money_box1 p{ font-size: 28px;}
    .money_box1 .title1{ margin-top: 10px;}
    .sendTran{ width: 1050px; margin: 0 auto; padding-left: 60px; padding-top: 30px; }
    .sendTran1{ width: 600px; }
    .inputBox{ margin-top: 8px; }
    .inputBox label{ color: #505558; font-size: 17px;  }
    .inputBox .inputText{ 
        margin-top: 5px;
        width: 100%;
        border: 1px solid #bdbfc0;
        color: #3a3f43;
        padding: 0.35em 1em 0.3166em 1em;
        border-radius: 3px; 
        height:30px; 
        line-height: 30px;
    }
    .historyBox{
         margin-top: 50px;  
    }
    .historyBox1{
        width: 1000px;
        margin: 0 auto;
    }
</style>
