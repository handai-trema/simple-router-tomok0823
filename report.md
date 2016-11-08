#レポート課題(11/2出題)

##課題内容

```
ルータのコマンドラインインタフェース (CLI) を作ろう。

次の操作ができるコマンドを作ろう。

 ・ルーティングテーブルの表示
 ・ルーティングテーブルエントリの追加と削除
 ・ルータのインタフェース一覧の表示
 ・そのほか、あると便利な機能
```

##解答

###サブコマンドの実装
本課題のコマンドを実装するために， ```/bin/simple_router``` のバイナリファイルを自分で作成した．このファイルに，各コマンドについての仕様を記述した．

###１. ルーティングテーブルの表示
- ```/bin/simple_router``` に以下に示す内容を記述した．

```
  command :show_table do |c|
    c.desc 'Location to find socket files'
    c.flag [:S, :socket_dir], default_value: Trema::DEFAULT_SOCKET_DIR

    c.action do |_global_options, options, args|
      print "\nShow routing table\n\n"
      print "destination/netmask_length\t next_hop\n"
      @routing_table = Trema.trema_process('SimpleRouter', options[:socket_dir]).controller.show_table()

      @routing_table.each do |each|
        each.each_key do |key, value|
          print IPv4Address.new(key).to_s,"/",@routing_table.index(each).to_s,"\t\t\t",each[key].to_s,"\n"
        end
      end
    end
  end
```


- ```/lib/simple_router.rb``` に以下に示すメソッド ```show_table``` を追加した．

```
  def show_table()
    return @routing_table.show()
  end
```

また，```/lib/routing_table``` に以下に示すメソッド ```show``` を追加した．

```
  def show()
    return @db
  end
```

```show_table``` メソッドでは，RoutingTableクラス内の ```show``` メソッドにおける返り値 ```@db``` を返り値として返す処理を行う．  
そして，```/bin/simple_router``` において，```show_table``` メソッドによって返ってきたルーティングテーブルの内容を表示する処理を行うコマンド ```show_table``` を実装した．  

###実行結果
```simple_router.conf``` で定義されている初期状態のルーティングテーブルの内容を以下に示す．

```
  ROUTES = [
    {
      destination: '0.0.0.0',
      netmask_length: 0,
      next_hop: '192.168.1.2'
    }
  ]
```

ルーターを起動し，実際に作成したコマンド ```show_table``` を実行した結果は以下のようになった．
```
ensyuu2@ensyuu2-VirtualBox:~/simple-router-tomok0823$ ./bin/simple_router show_table

Show routing table

destination/netmask_length	 next_hop
0.0.0.0/0			192.168.1.2
```

以上より，実装したルーティングテーブルの内容を出力するコマンド ```show_table``` が正しく動作することが確認できた．

###未完成