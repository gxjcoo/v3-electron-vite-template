<template>
  <div class="serial-component">
    <table width="950" height="423" align="center">
      <tbody>
        <tr>
          <td width="120" height="50">
            <button @click="SelectSerial" style="width: 100%">选择串口</button>
          </td>
          <td width="800">
            波特率:
            <select v-model="baudRate">
              <option v-for="rate in baudRates" :key="rate" :value="rate">{{ rate }}</option>
            </select>
            &nbsp;数据位:
            <select v-model="dataBits">
              <option v-for="bit in dataBitsOptions" :key="bit" :value="bit">{{ bit }}</option>
            </select>
            &nbsp;停止位:
            <select v-model="stopBits">
              <option v-for="stop in stopBitsOptions" :key="stop" :value="stop">{{ stop }}</option>
            </select>
            &nbsp;校验位:
            <select v-model="parity">
              <option v-for="p in parityOptions" :key="p" :value="p">{{ p }}</option>
            </select>
            &nbsp;
            <!-- 当已经选定串口后显示打开/关闭按钮 -->
            <button v-if="ports" @click="OpenSerial" style="width: 80px">打开串口</button>
            <button v-if="ports" @click="CloseSerial" style="width: 80px">关闭串口</button>
          </td>
        </tr>
        <tr>
          <td height="36">
            <button @click="beep" style="width: 100%">驱动发卡器响声</button>
          </td>
          <td>
            响声延时:
            <input
              type="text"
              v-model="beepDelay"
              size="5"
              maxlength="4"
              @input="beepDelay = beepDelay.replace(/\D/g, '')"
              style="color: blue; text-align: center"
            />
            毫秒
          </td>
        </tr>
        <tr>
          <td height="36">
            <button @click="Request" style="width: 100%">仅读取M1卡序列号</button>
          </td>
          <td>
            <label style="color: blue">{{ labelDisp }}</label>
          </td>
        </tr>
        <tr>
          <td height="36">
            <button @click="ReadM1Card" style="width: 100%">读取M1卡扇区数据</button>
          </td>
          <td>
            扇区号:
            <select v-model="sectorIndex" style="color: blue">
              <option v-for="(s, index) in sectors" :key="index" :value="s">第{{ s }}扇区</option>
            </select>
            <select v-model="inOutKey" style="color: blue">
              <option>内部密钥认证</option>
              <option>外部密钥认证</option>
            </select>
            <select v-model="authMode" style="color: blue">
              <option>B密钥认证</option>
              <option>A密钥认证</option>
            </select>
            ，认证密钥:
            <input
              type="text"
              v-model="authKey"
              size="12"
              maxlength="12"
              @input="authKey = authKey.replace(/[^0-9a-fA-F]/g, '')"
              style="color: blue; text-align: center"
            />
          </td>
        </tr>
        <tr>
          <td height="69">
            <button @click="WriteM1Card" style="width: 100%">写数据到M1卡扇区</button>
          </td>
          <td>
            <textarea v-model="RWM1Data" style="width: 800px; color: red" cols="100" rows="4" />
          </td>
        </tr>
        <tr>
          <td height="36">
            <button @click="changecardkeyex" style="width: 100%">修改M1卡扇区密钥</button>
          </td>
          <td>
            <select v-model="changeKeyOption" style="color: blue">
              <option>只修改A密钥</option>
              <option>只修改AB密钥</option>
              <option>修改AB密钥及控制位</option>
            </select>
            ，新A密钥
            <input
              type="text"
              v-model="newKeyA"
              size="12"
              maxlength="12"
              @input="newKeyA = newKeyA.replace(/[^0-9a-fA-F]/g, '')"
              style="color: red; text-align: center"
            />
            ，控制位
            <input
              type="text"
              v-model="cardCtr"
              size="8"
              maxlength="8"
              @input="cardCtr = cardCtr.replace(/[^0-9a-fA-F]/g, '')"
              style="color: red; text-align: center"
            />
            ，B密钥
            <input
              type="text"
              v-model="newKeyB"
              size="12"
              maxlength="12"
              @input="newKeyB = newKeyB.replace(/[^0-9a-fA-F]/g, '')"
              style="color: red; text-align: center"
            />
          </td>
        </tr>
        <tr>
          <td height="70">
            <p align="center">发送的数据</p>
          </td>
          <td>
            <textarea v-model="SendData" style="width: 800px; color: blue" cols="100" rows="4" />
          </td>
        </tr>
        <tr>
          <td height="70">
            <p align="center">接收的数据</p>
          </td>
          <td>
            <textarea v-model="ReceiveData" style="width: 800px" cols="100" rows="4" />
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: 'SerialComponent',
  data() {
    return {
      // 串口相关
      ports: [],
      port: null,
      reader: null,
      reading: false,
      // 用于接收数据
      getdata: new Uint8Array(1000),
      DataPoint: 0,
      SendCode: 0,
      SendData: '',
      ReceiveData: '',
      labelDisp: '',
      // 串口参数
      baudRate: 19200,
      baudRates: [
        1200, 4800, 9600, 14400, 19200, 38400, 43000, 57600, 115200, 128000, 230400, 256000, 460800,
        921600, 1382400
      ],
      dataBits: 8,
      dataBitsOptions: [8, 7, 6, 5],
      stopBits: 1,
      stopBitsOptions: [1, 1.5, 2],
      parity: 'None  无',
      parityOptions: ['None  无', 'Odd   奇', 'Even  偶', 'Mask  常1', 'Space 常0'],
      // 响声延时
      beepDelay: '30',
      // M1卡操作相关
      authKey: 'FFFFFFFFFFFF',
      RWM1Data: '',
      sectorIndex: 0,
      sectors: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15],
      inOutKey: '外部密钥认证',
      authMode: 'A密钥认证',
      // 修改卡密钥相关
      newKeyA: 'FFFFFFFFFFFF',
      cardCtr: 'FF078069',
      newKeyB: 'FFFFFFFFFFFF',
      changeKeyOption: '修改AB密钥及控制位',
      // 常量定义
      BLOCK0_EN: 0x01,
      BLOCK1_EN: 0x02,
      BLOCK2_EN: 0x04,
      NEEDSERIAL: 0x08,
      EXTERNKEY: 0x10,
      NEEDHALT: 0x20
    }
  },
  mounted() {},
  methods: {
    isUIntNum(val) {
      return /^\d+$/.test(val)
    },
    isHex(val) {
      return /^([0-9A-Fa-f])+$/.test(val)
    },
    async SelectSerial() {
      const port = await window.electron.ipcRenderer.invoke('ports')
      console.log(port, 'ports-11')

      this.port = port
    },
    updateInputData(data) {
      const array = new Uint8Array(data)
      for (const byte of array) {
        this.getdata[this.DataPoint] = byte
        this.DataPoint++
      }
      let crc = 0
      for (let i = 1; i < this.DataPoint; i++) {
        crc ^= this.getdata[i]
      }
      if (crc === 0 && this.DataPoint > 1) {
        let hexstr = ''
        for (let i = 0; i < this.DataPoint; i++) {
          hexstr += this.getdata[i].toString(16).padStart(2, '0').toUpperCase() + ' '
        }
        this.ReceiveData += hexstr
        let dispstr = ''
        let cardnohex = ''
        let datahex = ''
        switch (this.SendCode) {
          case 1:
            break
          case 2:
          case 3:
          case 4:
          case 5:
            if (this.getdata[0] === 1) {
              switch (this.getdata[1]) {
                case 8:
                  dispstr = '未寻到卡！'
                  break
                case 9:
                  dispstr = '两张以上卡片同时在感应区，发生冲突！'
                  break
                case 10:
                  dispstr = '无法选择激活卡片！'
                  break
                case 11:
                  dispstr = '密码装载失败，卡片序列号已知！'
                  break
                default:
                  dispstr = '操作卡失败，返回代码：' + this.getdata[1].toString()
                  break
              }
              this.labelDisp = dispstr
            } else if (this.getdata[0] === 5) {
              for (let i = 2; i < 6; i++) {
                cardnohex += this.getdata[i].toString(16).padStart(2, '0').toUpperCase()
              }
              if (this.SendCode === 2) {
                dispstr = '读卡序列号'
              } else if (this.SendCode === 3) {
                dispstr = '读扇区数据'
              } else if (this.SendCode === 4) {
                dispstr = '写扇区数据'
              } else if (this.SendCode === 5) {
                dispstr = '修改扇区密码'
              }
              switch (this.getdata[1]) {
                case 0:
                  dispstr += '成功！卡号：' + cardnohex
                  break
                case 1:
                  dispstr += '，密码认证成功,但读写扇区内容失败！卡号：' + cardnohex
                  break
                case 2:
                  dispstr +=
                    '，第0块操作成功，但第1、2块操作失败，仅扇区内容前16个字节的数据有效！！卡号：' +
                    cardnohex
                  break
                case 3:
                  dispstr +=
                    '，第0、1块操作成功，但第2块操作失败，仅扇区内容前32个字节的数据有效！！卡号：' +
                    cardnohex
                  break
                case 12:
                  dispstr += '，密码认证失败，卡号：' + cardnohex
                  break
                default:
                  dispstr += '，操作卡失败，返回代码：' + this.getdata[1].toString()
                  break
              }
              this.labelDisp = dispstr
            } else if (this.getdata[0] === 53) {
              for (let i = 2; i < 6; i++) {
                cardnohex += this.getdata[i].toString(16).padStart(2, '0').toUpperCase()
              }
              dispstr = '读M1卡扇区数据成功，卡号：' + cardnohex
              for (let i = 6; i < this.DataPoint - 1; i++) {
                datahex += this.getdata[i].toString(16).padStart(2, '0').toUpperCase() + ' '
              }
              this.labelDisp = dispstr
              this.RWM1Data = datahex
            } else if (this.getdata[0] === 54) {
              for (let i = 2; i < 6; i++) {
                cardnohex += this.getdata[i].toString(16).padStart(2, '0').toUpperCase()
              }
              dispstr = '读M1卡扇区数据成功，卡号：' + cardnohex
              for (let i = 7; i < this.DataPoint - 1; i++) {
                datahex += this.getdata[i].toString(16).padStart(2, '0').toUpperCase() + ' '
              }
              this.labelDisp = dispstr
              this.RWM1Data = datahex
            }
            break
        }
        this.DataPoint = 0
      }
    },
    async listenReceived() {
      if (this.reading) {
        console.log('On reading.')
        return
      }
      this.reading = true
      while (this.port.readable && this.reading) {
        this.reader = this.port.readable.getReader()
        try {
          const { value, done } = await this.reader.read()
          if (done) break
          this.updateInputData(value)
        } catch (e) {
          alert(e)
        } finally {
          this.reader.releaseLock()
        }
      }
      await this.port.close()
      this.port = null
      alert('串口已关闭！')
    },
    async OpenSerial() {
      if (!this.ports) {
        alert('请先选择要操作的串口号！')
        return
      }
      // 此处只传递了 baudRate，若需要可扩展传入 dataBits、stopBits、parity 等参数
      this.port = await window.electron.ipcRenderer.invoke('openPort')
      console.log(this.port, 'open-port')
      // this.listenReceived()
      // alert('串口打开成功！')
    },
    async CloseSerial() {
      if (!this.port || !this.port.writable) {
        alert('请选择并打开与发卡器相连的串口！')
        return
      }
      if (this.reading) {
        this.reading = false
        if (this.reader) await this.reader.cancel()
      }
    },
    async beep() {
      if (!this.port || !this.port.writable) {
        alert('请选择并打开与发卡器相连的串口！')
        return
      }
      const beepdelay = parseInt(this.beepDelay)
      const outputData = new Uint8Array(5)
      outputData[0] = 0x03
      outputData[1] = 0x0f
      outputData[2] = beepdelay % 256
      outputData[3] = Math.floor(beepdelay / 256)
      outputData[4] = outputData[1] ^ outputData[2] ^ outputData[3]
      let sendhex = ''
      for (let i = 0; i < 5; i++) {
        sendhex += outputData[i].toString(16).padStart(2, '0').toUpperCase() + ' '
      }
      this.SendData = sendhex
      this.ReceiveData = ''
      this.SendCode = 1
      this.DataPoint = 0
      const writer = this.port.writable.getWriter()
      await writer.write(outputData)
      writer.releaseLock()
    },
    async Request() {
      if (!this.port || !this.port.writable) {
        alert('请选择并打开与发卡器相连的串口！')
        return
      }
      const outputData = new Uint8Array(3)
      outputData[0] = 0x01
      outputData[1] = 0xf0
      outputData[2] = 0xf0
      let sendhex = ''
      for (let i = 0; i < 3; i++) {
        sendhex += outputData[i].toString(16).padStart(2, '0').toUpperCase() + ' '
      }
      this.SendData = sendhex
      this.ReceiveData = ''
      this.SendCode = 2
      this.DataPoint = 0
      this.labelDisp = ''
      const writer = this.port.writable.getWriter()
      await writer.write(outputData)
      writer.releaseLock()
    },
    async ReadM1Card() {
      if (!this.port || !this.port.writable) {
        alert('请选择并打开与发卡器相连的串口！')
        return
      }
      const myctrlword =
        this.inOutKey === '外部密钥认证'
          ? this.BLOCK0_EN + this.BLOCK1_EN + this.BLOCK2_EN + this.EXTERNKEY
          : this.BLOCK0_EN + this.BLOCK1_EN + this.BLOCK2_EN
      const myareano = this.sectorIndex
      const authmode = this.authMode === 'A密钥认证' ? 1 : 0
      const mypicckey = this.authKey.trim()
      if (!this.isHex(mypicckey) || mypicckey.length !== 12) {
        alert('认证密钥输入错误，请输入正确的12位16进制认证密钥！')
        return
      }
      const outputData = new Uint8Array(16)
      outputData[0] = 0x0e
      outputData[1] = 0x78
      outputData[2] = myctrlword
      outputData[3] = 0x00
      outputData[4] = 0x00
      outputData[5] = 0x00
      outputData[6] = 0x00
      outputData[7] = myareano
      outputData[8] = authmode
      for (let i = 0; i < 6; i++) {
        outputData[9 + i] = parseInt(mypicckey.substr(i * 2, 2), 16)
      }
      let crc = 0
      for (let i = 1; i < 15; i++) {
        crc ^= outputData[i]
      }
      outputData[15] = crc
      let sendhex = ''
      for (let i = 0; i < 16; i++) {
        sendhex += outputData[i].toString(16).padStart(2, '0').toUpperCase() + ' '
      }
      this.SendData = sendhex
      this.ReceiveData = ''
      this.RWM1Data = ''
      this.SendCode = 3
      this.DataPoint = 0
      this.labelDisp = ''
      const writer = this.port.writable.getWriter()
      await writer.write(outputData)
      writer.releaseLock()
    },
    async WriteM1Card() {
      if (!this.port || !this.port.writable) {
        alert('请选择并打开与发卡器相连的串口！')
        return
      }
      const myctrlword =
        this.inOutKey === '外部密钥认证'
          ? this.BLOCK0_EN + this.BLOCK1_EN + this.BLOCK2_EN + this.EXTERNKEY
          : this.BLOCK0_EN + this.BLOCK1_EN + this.BLOCK2_EN
      const myareano = this.sectorIndex
      const authmode = this.authMode === 'A密钥认证' ? 1 : 0
      const mypicckey = this.authKey.trim()
      if (!this.isHex(mypicckey) || mypicckey.length !== 12) {
        alert('认证密钥输入错误，请输入正确的12位16进制认证密钥！')
        return
      }
      let mypiccdata = this.RWM1Data.trim().replace(/\s/g, '')
      if (this.isHex(mypiccdata)) {
        if (mypiccdata.length < 96) {
          if (confirm('写卡数据不足一扇区48个字节，是否要后面补0写入？')) {
            while (mypiccdata.length < 96) {
              mypiccdata += '0'
            }
          } else {
            return
          }
        }
      } else {
        alert('请输入96位16进制写卡数据！')
        return
      }
      const outputData = new Uint8Array(64)
      outputData[0] = 0x3e
      outputData[1] = 0x69
      outputData[2] = myctrlword
      outputData[3] = 0x00
      outputData[4] = 0x00
      outputData[5] = 0x00
      outputData[6] = 0x00
      outputData[7] = myareano
      outputData[8] = authmode
      for (let i = 0; i < 6; i++) {
        outputData[9 + i] = parseInt(mypicckey.substr(i * 2, 2), 16)
      }
      for (let i = 0; i < 48; i++) {
        outputData[15 + i] = parseInt(mypiccdata.substr(i * 2, 2), 16)
      }
      let crc = 0
      for (let i = 1; i < 63; i++) {
        crc ^= outputData[i]
      }
      outputData[63] = crc
      let sendhex = ''
      for (let i = 0; i < 64; i++) {
        sendhex += outputData[i].toString(16).padStart(2, '0').toUpperCase() + ' '
      }
      this.SendData = sendhex
      this.ReceiveData = ''
      this.SendCode = 4
      this.DataPoint = 0
      this.labelDisp = ''
      const writer = this.port.writable.getWriter()
      await writer.write(outputData)
      writer.releaseLock()
    },
    async changecardkeyex() {
      if (!this.port || !this.port.writable) {
        alert('请选择并打开与发卡器相连的串口！')
        return
      }
      const myctrlword =
        this.inOutKey === '外部密钥认证'
          ? this.BLOCK0_EN + this.BLOCK1_EN + this.BLOCK2_EN + this.EXTERNKEY
          : this.BLOCK0_EN + this.BLOCK1_EN + this.BLOCK2_EN
      const myareano = this.sectorIndex
      const authmode = this.authMode === 'A密钥认证' ? 1 : 0
      const mypicckey = this.authKey.trim()
      if (!this.isHex(mypicckey) || mypicckey.length !== 12) {
        alert('认证密钥输入错误，请输入正确的12位16进制认证密钥！')
        return
      }
      const mypicckeyA = this.newKeyA.trim()
      if (!this.isHex(mypicckeyA) || mypicckeyA.length !== 12) {
        alert('新A密钥输入错误，请输入正确的12位16进制新A密钥！')
        return
      }
      const mypiccctr = this.cardCtr.trim()
      if (!this.isHex(mypiccctr) || mypiccctr.length !== 8) {
        alert('新控制位输入错误，请输入正确的8位16进制控制位！')
        return
      }
      const mypicckeyB = this.newKeyB.trim()
      if (!this.isHex(mypicckeyB) || mypicckeyB.length !== 12) {
        alert('新B密钥输入错误，请输入正确的12位16进制新B密钥！')
        return
      }
      let mypicckey_new = mypicckeyA + mypiccctr + mypicckeyB
      switch (this.changeKeyOption) {
        case '只修改A密钥':
          mypicckey_new += '00'
          break
        case '只修改AB密钥':
          mypicckey_new += '02'
          break
        default:
          mypicckey_new += '03'
          break
      }
      const outputData = new Uint8Array(33)
      outputData[0] = 0x1f
      outputData[1] = 0xf1
      outputData[2] = myctrlword
      outputData[3] = 0x00
      outputData[4] = 0x00
      outputData[5] = 0x00
      outputData[6] = 0x00
      outputData[7] = myareano
      outputData[8] = authmode
      for (let i = 0; i < 6; i++) {
        outputData[9 + i] = parseInt(mypicckey.substr(i * 2, 2), 16)
      }
      for (let i = 0; i < 17; i++) {
        outputData[15 + i] = parseInt(mypicckey_new.substr(i * 2, 2), 16)
      }
      let crc = 0
      for (let i = 1; i < 32; i++) {
        crc ^= outputData[i]
      }
      outputData[32] = crc
      let sendhex = ''
      for (let i = 0; i < 33; i++) {
        sendhex += outputData[i].toString(16).padStart(2, '0').toUpperCase() + ' '
      }
      this.SendData = sendhex
      this.ReceiveData = ''
      this.SendCode = 5
      this.DataPoint = 0
      this.labelDisp = ''
      const writer = this.port.writable.getWriter()
      await writer.write(outputData)
      writer.releaseLock()
    }
  }
}
</script>

<style scoped>
table {
  margin: 0 auto;
}
th {
  font-family: 楷体;
  background-color: #f6faff;
  color: blue;
}
td {
  font-family: 楷体;
  background-color: #f6faff;
}
</style>
