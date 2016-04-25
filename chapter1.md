# Replica , Consistency and hint

## Replica Factor

代表一份資料會在 Cluster 中儲存幾份，Replica Factor = 3 表示資料會存三分。通常就是負責該資料 Token 的 Node 及其相鄰的 Node 。

Cassandra 文件中每個 Node 都會有數字和顏色，那個是用來表示每個 Node 負責的 Token range，因此 RF 數量越高，代表每個 Node 負責的 Token range 比較大，同時各 Node 負責的 token 也會有重複的狀況。

Replica 也可以設定成跨資料中心使用，所以可以設定在不同的資料中心有不同的資料份數。

當資料寫入需求到達 Coordinator Node 時，Coordinator node 會根據資料的 token 與 Replica Factor 往對應的 Node 做非同步寫入，(疑問，那什麼時候才回應 Client 端寫入成功??)，同時如果有設定跨資料中心 RF 的話，也會往其他資料中心送資料，並交由另外一個資料中心做非同步寫入的動作。

經由適當的 Replica Factor 設定， Cassandra 可以保證資料安全 (機櫃配置應該也要注意吧，要是相鄰的 Node 都在同一櫃，甚至根本同一台的三個 VM 不就死人了) ，不管是死機或是火災燒機房。

## Consistency Level

代表資料在 Cassandra Cluster 中的一致性，這解決了上一節的何時才會回覆 Client 寫入或讀取成功的問題。
當資料寫入時，Cassandra 會根據 RF 設定往負責的 node 傳送資料，如果 RF = 3 ，當然最佳一致性就是三個 node 都回覆 coordinator node ，資料已經收到，寫 commit log, memtable 都成功了，

Consistency level 的表可以去查一下 [Datastax 的文件] (https://docs.datastax.com/en/cassandra/2.0/cassandra/dml/dml_config_consistency_c.html)，這邊講了幾種 Consistency level

假設你有一個由五個 node 組成的 Cassandra cluster


| Consitency Level | Replica Factor | Cluster 行為解釋                   |
| -----------------|----------------|----------------------------------------|
| One, Two, Three  | RF = 3         |  一個、兩個或三個 node 回覆寫入或讀取成功 |
| ALL              | RF = 3         | 全部 Replication node 寫入成功，在這個設定下是 3 node |
