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

###２. ルーティングテーブルエントリの追加と削除
ここでは，ルーティングテーブルエントリを追加するためのコマンド ```add_entry``` と，削除するためのコマンド ```delete_entry``` を実装した．
まず， ```/bin/simple_router``` において以下に示す ```add_entry``` コマンドを追加した．

```
  desc 'Adds a entry of routing table'
  arg_name 'destination_ip netmask_length next_hop'
  command :add_entry do |c|
    c.desc 'Location to find socket files'
    c.flag [:S, :socket_dir], default_value: Trema::DEFAULT_SOCKET_DIR

    c.action do |_global_options, options, args|
      destination_ip = args[0]
      netmask_length = args[1].to_i
      next_hop = args[2]
      Trema.trema_process('SimpleRouter', options[:socket_dir]).controller.
        add_entry(destination_ip, netmask_length, next_hop)
    end
  end
```

```add_entry``` コマンドでは，コマンドライン引数に順に ```destination_ip```, ``` netmask_length```, ```next_hop```を設定した．これにより，コマンド上でこれらの引数を入力すると ```SimpleRouter``` クラスの ```add_entry``` メソッドを呼び出しそれぞれの引数を与える．  
以下に，```add_entry``` メソッドの内容を記述する．

```
  def add_entry(destination_ip, netmask_length, next_hop)
    @routing_table.add({:destination => destination_ip, :netmask_length => netmask_length, :next_hop => next_hop})
  end
```

ここでは，```RoutingTable``` クラスで元々定義されている ```add``` メソッドにそれぞれの引数を与えて，ルーティングテーブルに指定された引数のエントリを追加する処理を行っている．  
次に，```/bin/simple_router``` に以下に示す ```delete_entry``` コマンドを追加した．

```
  desc 'Deletes a entry of routing table'
  arg_name 'destination_ip, netmask_length'
  command :delete_entry do |c|
    c.desc 'Location to find socket files'
    c.flag [:S, :socket_dir], default_value: Trema::DEFAULT_SOCKET_DIR
    c.action do |_global_options, options, args|
      destination_ip = args[0]
      netmask_length = args[1].to_i
      Trema.trema_process('SimpleRouter', options[:socket_dir]).controller.
        delete_entry(destination_ip, netmask_length)
    end
  end
```

ここでは，コマンドライン引数に ```destination_ip``` と ```netmask_length``` を指定し，この内容に当てはまるエントリをルーティングテーブルから削除するためのメソッド ```delete_entry``` を呼び出す．  
以下に，```delete_entry``` メソッドを示す．

```
  def delete_entry(destination_ip, netmask_length)
    @routing_table.delete_table({:destination => destination_ip, :netmask_length => netmask_length})
  end
```

ここでは，```add_entry``` メソッドの場合と同様に，エントリを削除するためのメソッドである```RoutingTable``` クラスの ```delete_table``` メソッドを呼び出す．

```
  def delete_table(options)
    netmask_length = options.fetch(:netmask_length)
    prefix = IPv4Address.new(options.fetch(:destination)).mask(netmask_length)
    @db[netmask_length].delete(prefix.to_i)
  end
```

このメソッドは，元々存在していた ```add``` メソッドを参考にして新たに作成した．指定したprefixに一致する要素を ```@db```から削除する処理を行う．

###実行結果
実際に，実装した ```add_entry``` コマンドと ```delete_entry``` コマンドを以下の手順で実行した．

```
０. 初期状態のテーブルの内容を表示
１. 適当なエントリをルーティングテーブルに追加
２. テーブルの内容を表示し追加を確認
３. 追加したエントリを削除
４. テーブルの内容を表示し削除を確認
```

以下に実行時の様子を示す．

```
ensyuu2@ensyuu2-VirtualBox:~/simple-router-tomok0823$ ./bin/simple_router show_table

Show routing table

destination/netmask_length	 next_hop
0.0.0.0/0			192.168.1.2
ensyuu2@ensyuu2-VirtualBox:~/simple-router-tomok0823$ ./bin/simple_router add_entry 192.168.1.10 24 192.168.1.2
ensyuu2@ensyuu2-VirtualBox:~/simple-router-tomok0823$ ./bin/simple_router show_table

Show routing table

destination/netmask_length	 next_hop
0.0.0.0/0			192.168.1.2
192.168.1.0/24			192.168.1.2
ensyuu2@ensyuu2-VirtualBox:~/simple-router-tomok0823$ ./bin/simple_router delete_entry 192.168.1.10 24
ensyuu2@ensyuu2-VirtualBox:~/simple-router-tomok0823$ ./bin/simple_router show_table

Show routing table

destination/netmask_length	 next_hop
0.0.0.0/0			192.168.1.2
```

以上により，正しくエントリの追加と削除が行えたことを確認できた．
