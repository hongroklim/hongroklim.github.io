---
date: 2022-08-18 17:00:00 +0900
title: NeoVim으로 Java 개발환경 구축하기
excerpt: 개발할 때 마우스 쓸 일이 점점 없어지는 중이다
categories: java
---

## 오랜만에 Java를 쓸 차례

Scala로 개발하다가 수업 과제로 Java를 써야 할 일이 생겼다. 그동안 NeoVim에다가
Metals 플러그인 설치해서 개발해 왔는데 ([관련 블로그]()), Java도 이와 비슷하게
개발할 수 있지 않을까 해서 찾아보았다.

언어 별로 어떤 LSP가 있는지 보여주는 [사이트](https://langserver.org)에서
[eclipse JDT Language Server](https://github.com/eclipse/eclipse.jdt.ls)를 찾을
수 있었다. NeoVim 클라이언트에 붙여 쓰기에는 여기 나온 설명으론 부족했기에
좀 더 찾아보았다. [nvim-jdtls](https://github.com/mfussenegger/nvim-jdtls)에
상세한 설명이 있어 여길 위주로 참고했다.

## Treesitter 설정

미리 설치해둔 [Treesitter](https://github.com/nvim-treesitter/nvim-treesitter)에
Java를 추가하기만 하면 된다.

```lua
require'nvim-treesitter.configs'.setup {
    // ...

    -- A list of parser names, or "all"
    ensure_installed = { "vim", "lua", "scala", "c", "scheme", "java" }

    // ...
}
```

그럼 NeoVim만 실행해도 알아서 설치된다.

## 기본 설정

### jdtls 설치

먼저 jdtls를 다운로드 받는다.

```sh
$ cd ./local/share/
$ wget https://download.eclipse.org/jdtls/milestones/1.14.0/jdt-language-server-1.14.0-202207211651.tar.gz
$ tar -xvzf jdt-language-server-1.14.0-202207211651.tar.gz
```

### 설정파일 위치

`README.md`에 따르면 Java를 실행할 때 자동으로 불러오도록 `ftplugin` 아래인
`~/.config/nvim/ftplugin/java.lua` 위치에 설정하라고 했다. 하지만 나는 Metals
설정을 `plugin`에 두고 동적으로 파일 타입 별로 불러오도록 설정했기에 
`~/.config/nvim/plugin/jdtls.lua`에다 위치를 잡았다.

### 변수 고치기

[Configuration 파일](https://github.com/mfussenegger/nvim-jdtls#configuration)을
그대로 복사한 뒤에 💀 표시 된 부분만 적절하게 바꿔주면 된다.

### Data Directory 설정

jdtls는 프로젝트 별로 폴더가 필요하다. 이는 `-data` 값에 지정해 주는데, 서로
겹치면 안 되므로 `workspace/` 아래 프로젝트의 root directory 이름을 따서
만들어준다.

```sh
# 먼저 workspace 폴더 만들기
$ mkdir -p ~/.local/share/jdt-language-server-1.14.0/workspace
```

```lua
-- 프로젝트 별 디렉토리를 생성한다
local root_markers = {'.classpath'}
local root_dir = require('jdtls.setup').find_root(root_markers)
local home = os.getenv('HOME')
local workspace_folder = home .. '/.local/share/jdt-language-server-1.14.0/workspace/' .. vim.fn.fnamemodify(root_dir, ':p:h:t')

local java_config = {
    -- ...
    cmd = {
        -- ...
        '-data', workspace_folder,
        -- ...
    }
    
    root_dir = root_dir,
    -- ...
}
```

### File Type 설정

Java 파일을 열었을 때 LSP를 실행하도록 설정할 차례이다.

```lua
api.nvim_create_autocmd("FileType", {
  pattern = { "java" },
  callback = function()
    require('jdtls').start_or_attach(java_config)
  end,
})
```

## Debugger 설정

NeoVim으로 Code Editing만 하기에는 아까우니 Debugger도 설정한다.


설치에 앞서 `JAVA_HOME`에 JAVA 17 이상이 설정되어 있어야 한다.
*(17이라니... 세월 참 덧없다)*

```sh
$ cd ~/.local/share
$ git clone --depth=1 https://github.com/microsoft/java-debug.git
$ cd java-debug
$ ./mvnw clean install
```

`jdtls.lua` 파일에 해당 debugger를 연결한다.

```lua
local bundles = {
  vim.fn.glob(home .. "/.local/share/java-debug/com.microsoft.java.debug.plugin/target/com.microsoft.java.debug.plugin-*.jar")
};

java_config.init_options = {
  bundles = bundles;
}
```

그 다음 [nvim-dap](https://github.com/mfussenegger/nvim-dap)도 설정한다.

```lua
java_config.on_attach = function(client, bufnr)
  -- With `hotcodereplace = 'auto' the debug adapter will try to apply code changes
  -- you make during a debug session immediately.
  -- Remove the option if you do not want that.
  require('jdtls').setup_dap({ hotcodereplace = 'auto' })
end
```

## Test 설정

Unit Test를 위한 설정도 당연히 필요하다.

```sh
$ cd ~/.local/share
$ git clone --depth=1 https://github.com/microsoft/vscode-java-test
$ cd vscode-java-test
$ npm install
$ npm run build-plugin
```

`jdtls.lua` 파일에서 debugger 설정의 연장선상으로 아래와 같이 설정한다.

```lua
local bundles = -- 위와 같다

vim.list_extend(bundles, vim.split(vim.fn.glob(home .. "/.local/share/vscode-java-test/server/*.jar"), "\n"))
java_config.init_options = {
  bundles = bundles;
}
```

## 단축키 설정

웬만한 단축키들은 LSP에 들어 있어서 그대로 쓰면 된다. 대신에 Java에 특화된
명령어들은 별도로 설정해야 한다.

```lua
local api = vim.api
local function map(mode, lhs, rhs, opts)
  local options = { noremap = true }
  if opts then
    options = vim.tbl_extend("force", options, opts)
  end
  api.nvim_set_keymap(mode, lhs, rhs, options)
end

-- 여기서 키 설정
map("n", "<leader>jt", [[<cmd>lua require"jdtls".test_class()<CR>]])
map("n", "<leader>jn", [[<cmd>lua require"jdtls".test_nearest_method()<CR>]])
```

이외에도 다양한 명령어들이 있다.

```vim
nnoremap <A-o> <Cmd>lua require'jdtls'.organize_imports()<CR>
nnoremap crv <Cmd>lua require('jdtls').extract_variable()<CR>
vnoremap crv <Esc><Cmd>lua require('jdtls').extract_variable(true)<CR>
nnoremap crc <Cmd>lua require('jdtls').extract_constant()<CR>
vnoremap crc <Esc><Cmd>lua require('jdtls').extract_constant(true)<CR>
vnoremap crm <Esc><Cmd>lua require('jdtls').extract_method(true)<CR>
```

## Vim 명령어 설정

Jdtls 관련 Vim 명령어들은 오로지 2개 밖에 없다. 다양한 명령어들을 사용하려면
아래와 같이 설정한다.

```lua
java_config.on_attach = function(client, bufnr)
  -- ...
  require('jdtls.setup').add_commands()
end
```

## 실행

모든 설정을 마쳤으면 이제 아무 Java 파일을 열어보자.

<figure>
  <img src="https://i.imgur.com/gZXlzG5.png"
       alt="jdtls-service-ready">
  <figcaption>왼쪽 아래에 ServiceReady가 보인다.</figcaption>
</figure>

`K`를 눌러 Hover를 실행해 보았다.

<figure>
  <img src="https://i.imgur.com/do91It6.png"
       alt="jdtls-hover">
  <figcaption>Hover까지 잘 된다.</figcaption>
</figure>

`:Jdt` 를 입력하고 `Tab`을 눌러보았다.

<figure>
  <img src="https://i.imgur.com/yyvaCZx.png"
       alt="jdtls-vim-command">
  <figcaption>Jdt 관련 Vim 설정들도 잘 인식된다.</figcaption>
</figure>
