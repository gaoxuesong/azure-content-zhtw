1. 在入口網站中，瀏覽至要連線閘道的虛擬網路。

2. 在 VNet 刀鋒視窗的 [設定] 中，按一下 [子網路] 以展開 [子網路] 刀鋒視窗。

3. 在 [子網路] 刀鋒視窗上，按一下頂端的 [+閘道子網路]。這會開啟 [新增子網路] 刀鋒視窗。子網路的 [名稱] 會自動填入 'GatewaySubnet' 這個值。為了讓 Azure 將此子網路視為閘道子網路，需要有這個值。

	![新增閘道子網路](./media/vpn-gateway-add-gwsubnet-rm-portal-include/addgwsubnet250.png)

4. 您可以視需要變更位址範圍 CIDR 區塊。請檢查您的設定的特定需求，以確認建議的 CIDR 區塊。

5. 按一下刀鋒視窗底部的 [確定] 以建立子網路。

<!---HONumber=AcomDC_0810_2016------>