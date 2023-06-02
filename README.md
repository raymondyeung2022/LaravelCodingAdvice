# Laravel Coding Advice
ドリームキャリアの研修グループに対するコーディングアドバイス

### 目的
以前の現場で学んだLaravelに対する知識を整理し、
できるだけ研修メンバーたちにアドバイスしたいと思うからです

### 原則
- 最短時間で理解できるように、読めやすいコードを書きましょう
- できるだけLaravelのAPIを使いましょう
- 重複使用のコードを2回以上書くことを控えましょう

### やってはいけないこと
ビジネスロジックの処理とバリデーションを全てコントローラーで行うこと

理由はコントローラーが肥大化しやすくなり、コードの整理が難しくなります
代わりにサービスクラスで行いましょう。

例:
コントローラークラスで:
```
public function index(Request $request)
{
    $artist_condition = [
        'published' => $request->published
    ];
    $data = $this->artist_service->list($artist_condition);
}
```

サービスクラスで:
```
public function list($artist_condition)
{
    $this->artist->where($artist_condition['published'], 1)->get();
}
```

入力フォームのバリデーションはリクエストクラスで行ってください 。

参考リンク: https://yama-itech.net/laravel-form-validation

(「フォームリクエストを使用する方法」のところ)

注意点としては、authorizeメソッドにtrueを返却してください。

### やるべきこと
1. コントローラークラスとサービスクラスに、定義するメソッドにPHPDocを付けてください:
    ```
    /**
    * {メソッドの概要を説明する}
    *
    * @param Request $request
    * @return $data
    */
    ```

    - @paramに{データタイプ} {引数名}というパターンを付けます
    - @returnに{変数名}というパターンを付けます
    - 基本的に@paramと@returnは大丈夫ですが、throwなど場合も付けてもいいです
    - コントローラーのコンストラクタにPHPDocを付けなくてもいいです

    参考リンク: https://prograshi.com/blog/param-return-in-php-docs/

2. できるだけPSRコーディングスタイルガイドに従ってコードを書きましょう
    - PSRはPHP Standards Recommendations(PHP標準勧告)
    - 主にPSR-1、PSR-12が重要です
    
    (PSR-2は2019/8/10に非推奨され、PSR-12が代替として推奨されるようになりました。)

    参考リンク:
    - https://www.ritolab.com/entry/91 (PSR-1)
    - https://www.ritolab.com/posts/208 (PSR-12)
    - https://qiita.com/Takashi_INOUE/items/41d9fedaad1ff338cada (PSR-2ですが、上のPSR-12より分かりやすい)

3. 特に理由がなければ、SQL文ではなくLaravelのeloquentを使いましょう。
    - 長いSQL文であれば、タイプミスやSQL文法エラーが行いやすいです。
    - ただし、サブクエリにSQL文を書けば大丈夫です。

4. 使わないnamespaceを削除しましょう。

5. モデルクラスでリレーションだけ使いましょう。Eloquentはサービスクラスで使うべきです。

6. データ更新前の存在チェックで、if文でチェックすることをやめましょう。
    - LaravelのeloquentにupdateOrCreate()というメソッドがあり、更新する前にデータの存在をチェックします
    - 参考リンク: https://qiita.com/miriwo/items/774348cd80a6c591eb1a

7. 404の処理は、できるだけ以下のeloquentのメソッドを使いましょう:
    - findOrFail(): 探すデータIDが存在しなければ404を返却されます
    - updateOrFail(): 更新する前にデータIDが存在しなければ404を返却されます。
    
    (updateOrFailはLaravel 8.58.0以降だけ使えますが、このプロジェクトは8.83.26なので大丈夫です)
    
    ただし、理由があればabortを使ってもいいです。
    
    参考リンク: https://ryuzan03.hatenablog.com/entry/2019/10/04/070000

8. 1つメソッドの中に、重複使用できるコードを2回以上書くことを控えましょう
    例:
    ```
    public function getAllArtists($language)
    {
        switch($language){
            case null:
                $artists = $this->where('is_artist_japanese_published', '1')->where('artist_masterpiece_number', '<>', 0)->get();
                break;
            case 'en':
                $artists = $this->where('is_artist_english_published', '1')->where('artist_masterpiece_number', '<>', 0)->get();
                break;
            case 'cn':
                $artists = $this->where('is_artist_chinese_published', '1')->where('artist_masterpiece_number', '<>', 0)->get();
                break;
        }
        return $artists;
    }
    ```
    ここの:
    ```
    $this->where({言語公開フラグ}, '1')->where('artist_masterpiece_number', '<>', 0)->get();
    ```
    を3回書きました。
    データ取得に条件を変更する時、三箇所のEloquent文を修正しなければなりません。

    しかし以下のように書けば:
    ```
    public function getAllArtists($language)
    {
        $published='';
        switch($language){
            case null:
                $published = 'is_artist_japanese_published';
                break;
            case 'en':
                $published = 'is_artist_english_published';
                break;
            case 'cn':
                $published = 'is_artist_chinese_published';
                break;
        }
        $artists = $this->where($published, '1')
            ->where('artist_masterpiece_number', '<>', 0)->get();
        return $artists;
    } 
    ```
    Eloquentを1回だけ書きましたので、データ取得に条件を変更する時、1回しか修正しません。
    
    この書き方の方が楽ではありませんか?
    
    参考リンク: https://ssaits.jp/promapedia/glossary/ciode-clone.html

9. 各コントローラーのメソッドの命名パターンは以下のリンクのように守りましょう。:

    https://webdevetc.com/blog/laravel-naming-conventions/#section_method-naming-conventions-in-controllers
    
    英語ですが難しい内容ではありません。

    サービスクラスは以下のように命名しましょう:
    | コントローラーのメソッド名  | サービスクラスのメソッド名 |
    | ------------- | ------------- |
    | index  | list  |
    | show  | read  |
    | create  | create  |
    | store  | create  |
    | edit  | update  |
    | update  | update  |
    | delete  | delete  |
    | destory  | delete  |

10. ルートを設定する時、引数渡す場合バリデーションを行ってください。例:
    ```
    Route::get('/artist-list/{artist_id}', [ArtistController::class,'artist_show'])->name('artist_show')->where('artist_id', '[0-9]+');
    ```
    正規表現のところを気を付けてください

11. コントローラーから引数をサービスクラスに送る時、別の配列を作成して送りましょう。理由は2つ:
    - 1つメソッドの引数は2つ以下がいいです (参考リンク: https://qiita.com/tadsan/items/c47eb327684530721e8a#%E5%BC%95%E6%95%B0%E3%81%AF2%E3%81%A4%E4%BB%A5%E4%B8%8B%E3%81%8C%E7%90%86%E6%83%B3)
    - たくさん引数を送りたいなら、配列でまとめられます(「やってはいけない」ところに参考してください)

12. composer.jsonを変更した場合、autoloadをリロードする必要があるため以下のコマンドの実行をしてください。
    また、他の作業者にもコマンドを実行してもらうよう周知してください。

    ```
    ■実行コマンド
    composer dump-autoload
    ```