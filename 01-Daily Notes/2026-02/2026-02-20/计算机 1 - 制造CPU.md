---
url: "[把CPU拆到只剩一个晶体管，带你从零看懂计算机底层原理！\\| 晶体管 / 锁存器 / 寄存器 / 数据总线 / 地址总线 / 汇编语言 / 周期\\_哔哩哔哩\\_bilibili](https://www.bilibili.com/video/BV1rVnFzKECS?spm_id_from=333.788.videopod.sections&vd_source=56499cc54ebd02db0ac739e485d74801)"
---
                                                               
# 怎么运行指令

[[attachments/7e361aa2d3c8d313c78d75f4ccad3c88_MD5.png|Open: Pasted image 20260220091030.png]]
![[attachments/7e361aa2d3c8d313c78d75f4ccad3c88_MD5.png]]

[[attachments/1ba6904aab04b8ad4b0f0823a8daa37b_MD5.png|Open: Pasted image 20260220091136.png]]
![[attachments/1ba6904aab04b8ad4b0f0823a8daa37b_MD5.png]]

[[attachments/ef0101cfc2a773b645aca97afe709c16_MD5.png|Open: Pasted image 20260220091544.png]]
![[attachments/ef0101cfc2a773b645aca97afe709c16_MD5.png]]

[[attachments/3e48057c4147b4b79b3789399a40e2ae_MD5.png|Open: Pasted image 20260220091834.png]]
![[attachments/3e48057c4147b4b79b3789399a40e2ae_MD5.png]]

# 控制信号来自哪里

[[attachments/de419ae7eee5b564947ddae88d2cc676_MD5.png|Open: Pasted image 20260220092206.png]]
![[attachments/de419ae7eee5b564947ddae88d2cc676_MD5.png]]

[[attachments/3e5ace77228f6b6030381f4d04390569_MD5.png|Open: Pasted image 20260220092908.png]]
![[attachments/3e5ace77228f6b6030381f4d04390569_MD5.png]]

[[attachments/9b67d3ff0fc44bb7ac0cc1b586a59264_MD5.png|Open: Pasted image 20260220092915.png]]
![[attachments/9b67d3ff0fc44bb7ac0cc1b586a59264_MD5.png]]

[[attachments/4a45a0b6f9bca6275b4c6d1ea24eb143_MD5.png|Open: Pasted image 20260220092920.png]]
![[attachments/4a45a0b6f9bca6275b4c6d1ea24eb143_MD5.png]]

[[attachments/bed1402915534089c45cab494e0ff63e_MD5.png|Open: Pasted image 20260220093007.png]]
![[attachments/bed1402915534089c45cab494e0ff63e_MD5.png]]

## 实现加载指令

[[attachments/162594d15af23246b6a981898ea676a7_MD5.png|Open: Pasted image 20260220094547.png]]
![[attachments/162594d15af23246b6a981898ea676a7_MD5.png]]

[[attachments/2c60f11a4369a894f6c539e01f9bacad_MD5.png|Open: Pasted image 20260220095210.png]]
![[attachments/2c60f11a4369a894f6c539e01f9bacad_MD5.png]]

R1 选择了寄存器,既然选择了那么就是要对该寄存器进行读写,所以用来激活使能信号

读写都会被激活,通过指令的那个译码器输出进行过滤就可以了

[[attachments/c30889c2c16a10ccd598060eb55f7fca_MD5.png|Open: Pasted image 20260220095439.png]]
![[attachments/c30889c2c16a10ccd598060eb55f7fca_MD5.png]]

[[attachments/e759efe7727513288478cc78256c128e_MD5.png|Open: Pasted image 20260220095537.png]]
![[attachments/e759efe7727513288478cc78256c128e_MD5.png]]

[[attachments/ddea03c14e686d392a7e4fe51454b337_MD5.png|Open: Pasted image 20260220095601.png]]
![[attachments/ddea03c14e686d392a7e4fe51454b337_MD5.png]]

[[attachments/5a44a545e990db7d0671dce123115ee1_MD5.png|Open: Pasted image 20260220100123.png]]
![[attachments/5a44a545e990db7d0671dce123115ee1_MD5.png]]

[[attachments/09d5e5b5c4357a804a6088ff8bd8777f_MD5.png|Open: Pasted image 20260220100351.png]]
![[attachments/09d5e5b5c4357a804a6088ff8bd8777f_MD5.png]]

[[attachments/33bd8157cd8cfd598a4d814701d90065_MD5.png|Open: Pasted image 20260220100420.png]]
![[attachments/33bd8157cd8cfd598a4d814701d90065_MD5.png]]

# 指令寄存器

[[attachments/f7143500ce552a9113f75e404a878982_MD5.png|Open: Pasted image 20260220100500.png]]
![[attachments/f7143500ce552a9113f75e404a878982_MD5.png]]

## 关于优化的简短讨论

[[attachments/928db1c5bbd06db00a6dae8f02aee5d0_MD5.png|Open: Pasted image 20260220101020.png]]
![[attachments/928db1c5bbd06db00a6dae8f02aee5d0_MD5.png]]

[[attachments/e1e58986beed40f762d5af13dc935a77_MD5.png|Open: Pasted image 20260220101034.png]]
![[attachments/e1e58986beed40f762d5af13dc935a77_MD5.png]]

[[attachments/7fdbd531e8c8c6d49fcd1c84f1850db5_MD5.png|Open: Pasted image 20260220101115.png]]
![[attachments/7fdbd531e8c8c6d49fcd1c84f1850db5_MD5.png]]

[[attachments/78f89bb8fecc6f40642ae2743576763c_MD5.png|Open: Pasted image 20260220101133.png]]
![[attachments/78f89bb8fecc6f40642ae2743576763c_MD5.png]]

## 加法和减法的具体执行

[[attachments/6950226c8b20fcad078e51f99c41421c_MD5.mp4|Open: PixPin_2026-02-20_10-19-36.mp4]]
![[attachments/6950226c8b20fcad078e51f99c41421c_MD5.mp4]]

[[attachments/47bfac238eb4e3924af472c387bd72c8_MD5.png|Open: Pasted image 20260220102525.png]]
![[attachments/47bfac238eb4e3924af472c387bd72c8_MD5.png]]

[[attachments/95cab08be5baf86451549ebdc43605e2_MD5.png|Open: Pasted image 20260220103258.png]]
![[attachments/95cab08be5baf86451549ebdc43605e2_MD5.png]]

>![[../2026-02-18/attachments/f2838fb61b999cd9ed08fed256dce663_MD5.png]]

[[attachments/0fe458509e36477071016caef691785c_MD5.png|Open: Pasted image 20260220103929.png]]
![[attachments/0fe458509e36477071016caef691785c_MD5.png]]

[[attachments/d018b0e0fd8c83e91c0f35c07a6773ae_MD5.png|Open: Pasted image 20260220103944.png]]
![[attachments/d018b0e0fd8c83e91c0f35c07a6773ae_MD5.png]]

# 控制单元

[[attachments/3a7acc685b18da03be65ae25719a2929_MD5.png|Open: Pasted image 20260220104028.png]]
![[attachments/3a7acc685b18da03be65ae25719a2929_MD5.png]]

[[attachments/ca32c5c237706c20ff99306684e0e134_MD5.png|Open: Pasted image 20260220105145.png]]
![[attachments/ca32c5c237706c20ff99306684e0e134_MD5.png]]

[[attachments/74c31e673df3d5ee94259d801d060b25_MD5.png|Open: Pasted image 20260220105329.png]]
![[attachments/74c31e673df3d5ee94259d801d060b25_MD5.png]]

[[attachments/98c2da6d49cb333c0477b36de5474da9_MD5.png|Open: Pasted image 20260220105406.png]]
![[attachments/98c2da6d49cb333c0477b36de5474da9_MD5.png]]

[[attachments/435aba9fb2fe9bb5b1ed8b09511100db_MD5.png|Open: Pasted image 20260220105442.png]]
![[attachments/435aba9fb2fe9bb5b1ed8b09511100db_MD5.png]]

[[attachments/3a07448f8ae7109379965196e649f899_MD5.png|Open: Pasted image 20260220105543.png]]
![[attachments/3a07448f8ae7109379965196e649f899_MD5.png]]

[[attachments/c0bb02c07ba0c32f28b5873ed679d2e0_MD5.png|Open: Pasted image 20260220105656.png]]
![[attachments/c0bb02c07ba0c32f28b5873ed679d2e0_MD5.png]]

[[attachments/4bca4d8c1c5498b76e4bab014a5fc55a_MD5.png|Open: Pasted image 20260220105806.png]]
![[attachments/4bca4d8c1c5498b76e4bab014a5fc55a_MD5.png]]

## 取指阶段

[[attachments/b12b42600aa36fa4ec71b7b14167c4a5_MD5.png|Open: Pasted image 20260220105832.png]]
![[attachments/b12b42600aa36fa4ec71b7b14167c4a5_MD5.png]]

[[attachments/5ee6efbbf836eb76d589a374f6b14281_MD5.png|Open: Pasted image 20260220105924.png]]
![[attachments/5ee6efbbf836eb76d589a374f6b14281_MD5.png]]

## 译码阶段

[[attachments/9a604a30a59fd342ff8a87308d5478a1_MD5.png|Open: Pasted image 20260220110417.png]]
![[attachments/9a604a30a59fd342ff8a87308d5478a1_MD5.png]]

[[attachments/aee4232567e215d05af03533aecfca61_MD5.png|Open: Pasted image 20260220110445.png]]
![[attachments/aee4232567e215d05af03533aecfca61_MD5.png]]

[[attachments/efca8eeb97aa9ab7542a76a75230f07f_MD5.png|Open: Pasted image 20260220110521.png]]
![[attachments/efca8eeb97aa9ab7542a76a75230f07f_MD5.png]]

[[attachments/b6b015a7d66112aeedf4c338a33fadf1_MD5.png|Open: Pasted image 20260220110855.png]]
![[attachments/b6b015a7d66112aeedf4c338a33fadf1_MD5.png]]

[[attachments/f02636c11fbb894d86fb19051fdef07e_MD5.png|Open: Pasted image 20260220110933.png]]
![[attachments/f02636c11fbb894d86fb19051fdef07e_MD5.png]]

## 执行阶段

[[attachments/768d5ba6bc43a20dc7e569ac98b2a709_MD5.png|Open: Pasted image 20260220112553.png]]
![[attachments/768d5ba6bc43a20dc7e569ac98b2a709_MD5.png]]

[[attachments/95bdb6e5cf1ddc3c5d8a6b43168f4716_MD5.png|Open: Pasted image 20260220112650.png]]
![[attachments/95bdb6e5cf1ddc3c5d8a6b43168f4716_MD5.png]]

[[attachments/a2e71b966d33bee0e16095d4fb4c05c8_MD5.png|Open: Pasted image 20260220112735.png]]
![[attachments/a2e71b966d33bee0e16095d4fb4c05c8_MD5.png]]