## 累加部署與完整部署

資源管理員預設會將部署處理為資源群組的累加式更新。使用累加部署，資源管理員將：

- **保留且不變更**現存於資源群組中但未在範本中指定的資源
- **加入**在範本中指定但未存在於資源群組中的資源
- **不重新佈建**現存於資源群組，並在範本中以相同條件定義的資源
- **重新佈建**在範本中具有更新設定的現有資源

使用完整部署，資源管理員將：

- **刪除**現存於資源群組中但未在範本中指定的資源
- **加入**在範本中指定但未存在於資源群組中的資源
- **不重新佈建**現存於資源群組，並在範本中以相同條件定義的資源
- **重新佈建**在範本中具有更新設定的現有資源
 
您可以透過 **Mode** 屬性指定部署類型。

<!---HONumber=AcomDC_0713_2016-->