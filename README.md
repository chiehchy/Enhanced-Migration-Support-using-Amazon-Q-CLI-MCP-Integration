
# Enhanced Migration Support using Amazon Q CLI MCP Integration


### Outline

本Lab 通過 Amazon Q CLI + MCP 展示如何將地端的環境遷移到AWS


[通過 Amazon Q CLI 遷移DB 到AWS 託管的RDS MySQL](https://quip-amazon.com/xhmKAldRC2sr/Lab-Amazon-Q-CLI-MySQL-AWS-RDS-MySQL#temp:C:CDDec05ab16c86b466791658d64f)

* [給予EC2 所使用的IAM Role admin 權限](https://quip-amazon.com/xhmKAldRC2sr/0822-Workshop#temp:C:CDDb448925c441841ed8cddaafde)
* [安裝本地MySQL](https://quip-amazon.com/xhmKAldRC2sr/0822-Workshop#temp:C:CDD66ac53b192774b8c87486ef33)
* [安裝雲端RDS MySQL](https://quip-amazon.com/xhmKAldRC2sr/0822-Workshop#temp:C:CDD39499f6fa6cf48c8a40b59de8)
* [從地端將MySQL 資料庫遷移到雲端](https://quip-amazon.com/xhmKAldRC2sr/0822-Workshop#temp:C:CDDc4460d357c0242e1a46b6e592)



### （Optional）前置作業

1. 透過Q分析RVtool 的檔案收集：[RVTools_export_all_2025-02-06_19.36.35.zip](https://quip-amazon.com/-/blob/CDD9AAMtVK5/HzwMtWdyfmSXMrhwNxiIJQ?name=RVTools_export_all_2025-02-06_19.36.35.zip&s=xhmKAldRC2sr)  
    RVTool檔案下載連結：https://ws-assets-prod-iad-r-iad-ed304a55c2ca1aee.s3.us-east-1.amazonaws.com/72934c0e-28a4-4427-bfd1-417e46d678b2/config-scripts/RVTools_export_all_2025-02-06_19.36.35.zip
    RVTool檔案短網址：
    https://mistertanuki.blogspot.com/2018/08/gather-vmware-vsphere-info-with-rvtools.html
    *RVTools* 除了可以查看vSphere 的資訊，並可匯出保存之外，也可以觀察一些VM 的不正常使用狀況，算是幫助管理者的一個小工具
2. 使用Prompt: 分析RVTools資料夾中的檔案，規劃搬遷至雲上Amazon Elastic VMware Service (Amazon EVS)的計畫，資料庫的部分改換使用RDS MySQL，並產生架構圖
3. 生成資料庫規格檔：[vmware-to-aws-migration-plan.md](https://quip-amazon.com/-/blob/CDD9AAMtVK5/ns30n2_pWtl5DYyt5-gwDA?name=vmware-to-aws-migration-plan.md&s=xhmKAldRC2sr) 



### Q developer 設定注意事項：



```
/profile 換成 /agent

Commands:
  list         List all available agents
  create       Create a new agent with the specified name
  schema       Show agent config schema
  set-default  Define a default agent to use when q chat launches
  help         Print this message or the help of the given subcommand(s)

```



```
#幫助指令
/agent --help

#列出所有的Agent 
/profile list -> /agent list 

#創建user起始配置
/profile create dev -> /agent create --name dev

#下次新開Terminal啟用Q才會使用這個設定
/profile set default -> /agent set-default --name dev

/agent schema

```



```
#起始使用設定
q chat --agent documentation-profile

q mcp list
q mcp status
q mcp status --name XXXXX
```


登記使用的3個mcp server 可以幫助完成RVTools的分析，

[Image: image.png]
[Image: image.png]
[Image: image.png]
[Image: image.png]
[Image: image.png]




### Amazon Q CLI 智能遷移地端MySQL 資料庫到AWS RDS MySQL

1. 給予EC2 所使用的IAM Role admin 權限

* 在控制台找到 **EC2**

<img width="989" height="391" alt="image (32)" src="https://github.com/user-attachments/assets/128bc5df-c09a-401a-9be7-c5a1d7cde219" />

* 在左側選擇Instances → code-server-stack-CodeServer → Security → 點擊IAM Role 

<img width="1352" height="520" alt="image (33)" src="https://github.com/user-attachments/assets/53effc0f-ff76-45f9-a51c-66c11510378f" />


* 在右邊選擇 Add permissions → Attach policies →. AdminstratorAceess → Add permission
<img width="1350" height="547" alt="image (34)" src="https://github.com/user-attachments/assets/08104eaf-a775-4b52-b1a2-4a3691680f4d" />

<img width="1318" height="249" alt="image (35)" src="https://github.com/user-attachments/assets/87d5790e-d358-4f29-aeea-c5c005440f83" />

<img width="1546" height="210" alt="image (36)" src="https://github.com/user-attachments/assets/2707f16a-1a9a-4351-a2ea-61c795bf2e57" />



2. 在本地安裝MySQL



* 在Terminall 輸入Q 啟動Amazon Q, 接著輸入下面Prompt

幫我在本機安裝Community MySQL database, 並幫我設定帳號密碼為 root 和 password 123 , 安裝employee 範例資料庫

<img width="927" height="413" alt="image (37)" src="https://github.com/user-attachments/assets/941974a0-d9b1-48f6-aca3-a036b9d61965" />

<img width="829" height="269" alt="image (38)" src="https://github.com/user-attachments/assets/4d71b4b8-c71a-4b1f-b16b-2e2c20437cc8" />

<img width="966" height="596" alt="image (39)" src="https://github.com/user-attachments/assets/c3ddefd9-689f-453a-9933-b15f063d0898" />


* 開啟另外一個terminal 確認可以成功連接MySQL

<img width="706" height="403" alt="image (40)" src="https://github.com/user-attachments/assets/281de0d9-c485-43b9-a5d2-fa3eb47ef71d" />



3. 通過Amazon Q CLI 安裝RDS MySQL



* 在Q 下輸入下面Prompt 來安裝RDS MySQL

幫我在我的aws 帳號下安裝rds mysql8, 實例類型為t3.medium, 帳號密碼為 root 和 password 123, 要確保我的本地可以連上這台mysql

<img width="1037" height="663" alt="image (41)" src="https://github.com/user-attachments/assets/0cd0b1a5-f603-4319-ba9a-186de7abeeb4" />


* 開啟另外一個terminal 確認可以成功連接 RDS MySQL

<img width="1040" height="190" alt="image (42)" src="https://github.com/user-attachments/assets/e5fffb24-801e-4f3c-8ebf-1008792a3e7c" />


4. 使用Amazon Q CLI 遷移 MySQL  資料庫

* 在Amazon Q CLI 輸入下面Prompt, 其中RDS的連線信息需修改為前一步驟所創建的rds

```
幫我把本地MySQL 中的 employee 資料庫通過mysql dump遷移到雲端的RDS
本地mysql信息如下
host: 127.0.0.1
user: root
password : password123


雲端的mysql 信息如下
host: [mysql-rds-instance.cfyigeig6bx2.us-east-1.rds.amazonaws.com](http://mysql-rds-instance.cfyigeig6bx2.us-east-1.rds.amazonaws.com/)
user: root
password : password123
```

<img width="959" height="457" alt="image (43)" src="https://github.com/user-attachments/assets/44401089-70bf-450d-b389-052fff42e2d1" />







