---
title: Tailwindとsassの非互換問題の解決について
category: "Coding"
tags: [Ruby on Rails, css, tailwind, sass]
---

今回のエラーはGithub Actionsを使ったCI環境で出たエラー。
背景には、Tailwindとsassの非互換問題(incompatible)があることがわかった。

# 背景
`Tailwindcss-rails`の公式説明によると、TailwindがモダンなCSS構文を使っているけど、Sassが最新のCSS構文をまだ理解できていないため、アセットパイプラインでtailwindのcss構文を`sassc-rails`経由で圧縮したら、`SassC::SyntaxError`というエラーが出てしまう。tailwindを使うには、gem `sassc-rails` をアンインストールするのが必要だ。

エラー例：
```
SassC::SyntaxError: Error: Function rgb is missing argument $green.
        on line 1 of stdin
>> -white{--tw-bg-opacity:1;background-color:rgb(255 255 255/var(--tw-bg-opacity));
```
ここの`rgb(255 255 255/var(--tw-bg-opacity))`、つまり`rgb(0 0 0 / 0%)`形式の構文は最新のCSS構文で、通常の形式なら、
`rgb(0, 0, 0)`か、`rgba(0, 0, 0, 0)`にする必要。


`Tailwindcss-rails`の公式説明
>Tailwind uses modern CSS features that are not recognized by the sassc-rails extension that was included by default in the Gemfile for Rails 6. In order to avoid any errors like SassC::SyntaxError, you must remove that gem from your Gemfile.
https://github.com/rails/tailwindcss-rails#conflict-with-sassc-rails

しかし、今回のRails7アプリは、管理画面は`rails-admin`を使って構築した。`rails-admin`がまだSassに依存しているので、`sassc-rails`を削除したら、` cannot load such file -- sassc`エラーが出る。

# 解決方法
tailwindとsassを同時に使うため、CSS Compressorはsassでなく、別のモダンなcompressor(今回はCSSO)をカスタマイズして使用すること。

1. csso-cliをインストール
`yarn add csso-cli`

2. `config/initializers/csso.rb`ファイルを新規作成して、カスタムcompressor作成

```ruby
require "open3"

class CssoCompressor
  def self.call(input)
    puts "[CssoCompressor] Compressing…"

    # Copy the contents of the CSS file to a temp file 
    temp_file = Tempfile.new([input[:name], ".css"])
    temp_file.open
    temp_file.write(input[:data])
    temp_file.flush

    # Run the compressor and capture the output
    css, err, status = Open3.capture3("npx", "csso", temp_file.path)

    {data: css}
  end
end

Rails.application.config.assets.configure do |env|
  env.register_compressor "text/css", :csso, CssoCompressor
end
```

3. CSS Compressorを`:csso`に設定する  
`config.assets.css_compressor = :csso`  
CI環境でも使うため、`config/environments/`下のproduction.rbとtest.rb両方設定必要

設定参照：[create a custom compressor by domchristie ](https://github.com/rails/tailwindcss-rails/issues/82#issuecomment-1006779752)  

また、CI環境でtestを実行する前に、`precompile assets`のstepを追加する必要。
```
# Precompile assets
- name: Assets Precompile
  run: bundle exec rake assets:precompile
# Add or replace test runners here
- name: Run tests
  run: bundle exec rspec
```
じゃないと、`AssetNotFound`エラーが出る。
```
 ActionView::Template::Error:
   The asset "tailwind.css" is not present in the asset pipeline.
```

これでtailwindとsassの非互換問題が無事解決!

ちなみに、Rails7からはSass自体が非推奨になったようで、sassはなるべく使わない方が良いかも。

Rails6でデフォルトでインストールされていた`gem 'sassc-rails'`がRails7からデフォルトでコメントアウトされている。

>Modern web applications are more likely than not to use a CSS framework like Tailwind, Bootstrap, or Bulma. Rails shouldn't be generating per-model stylesheets as though we were still writing everything by hand.

>Also, Sass has chosen to focus exclusively on dart-sass, which requires all manner of dependencies that Rails won't adopt by default. So decrease our reliance on Sass, and move it to being an optional extra.

by DHH氏 [Remove default reliance on Sass and CSS generators #43110](https://github.com/rails/rails/pull/43110)


ほか参照：  
[assets:precompile results in LoadError: cannot load such file -- sassc](https://github.com/railsadminteam/rails_admin/issues/3450)  
[Heroku Issue: ActionView::Template::Error (The asset "tailwind.css" is not present in the asset pipeline.")](https://github.com/rails/tailwindcss-rails/issues/153#issuecomment-1191022279)
