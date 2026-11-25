请按照 https://github.com/rtl-workflow/rtl_doc_agent 的 README 中的提示，为我安装这个插件，并检查命令、
agent 和 skill 是否加载成功



https://125.67.215.202:9443
IP：125.67.215.202
端口：9443


VPN账号	VPN初始密码	服务器IP      	服务器账号	服务器初始密码
x0_user19	GK}~7:f7	10.0.2.243:19	localuser19	pass1234
SVN密码
85lMT6wn


jame:
mld_ext_guo：ArNEK8SK5l

" ==========================================
" 1. NERDTree 基础配置
" ==========================================

set hidden
set mouse=a
set showtabline=1
set guitablabel=%N\ %-0.35t%M

if has('gui_running')
    set guioptions+=e
endif

nnoremap gf <C-w>gf

set runtimepath+=~/.vim/bundle/nerdtree-master

let NERDTreeWinPos = "left"
let NERDTreeWinSize = 30
let NERDTreeQuitOnOpen = 0
let NERDTreeChDirMode = 0
let NERDTreeMouseMode = 2

" F3 显示/隐藏 NERDTree
nnoremap <F3> :NERDTreeToggle<CR>

" 每个Tab独立保存搜索内容
augroup TabLocalSearch
    autocmd!
    autocmd TabLeave * let t:last_search = @/
    autocmd TabEnter * let @/ = get(t:, 'last_search', '')
augroup END

" ==========================================
" 2. 每个 Tab 只初始化一次 NERDTree
" ==========================================

let g:nt_started = 0

function! s:InitNERDTree()

    " GVim 尚未完成启动
    if !g:nt_started
        return
    endif

    " 已初始化或正在初始化
    if get(t:, 'nt_ready', 0) || get(t:, 'nt_busy', 0)
        return
    endif

    " 跳过特殊 Buffer
    if &buftype !=# ''
        return
    endif

    " 获取当前文件路径
    let l:file = expand('%:p')

    if empty(l:file)
        return
    endif

    let l:dir = fnamemodify(l:file, ':h')

    if !isdirectory(l:dir)
        return
    endif

    let t:nt_busy = 1

    try
        " 创建当前 Tab 独立的 NERDTree
        execute 'NERDTree ' . fnameescape(l:dir)

        " 标记当前 Tab 已初始化
        let t:nt_ready = 1

        " 返回代码窗口
        if &filetype ==# 'nerdtree' && winnr('$') > 1
            wincmd p
        endif

    finally
        let t:nt_busy = 0
    endtry

endfunction


" ==========================================
" 3. 自动关闭只剩 NERDTree 的 Tab
" ==========================================

function! s:CloseLonelyNERDTree()

    " 当前 Tab 只剩一个窗口
    if winnr('$') != 1
        return
    endif

    " 判断唯一窗口是否为 NERDTree
    if exists('b:NERDTree')
        if b:NERDTree.isTabTree()
            quit
        endif
    endif

endfunction


" ==========================================
" 4. 自动命令
" ==========================================

augroup FastNERDTree
    autocmd!

    " GVim 启动后初始化目录树
    autocmd VimEnter * let g:nt_started = 1 |
                \ call <SID>InitNERDTree()

    " 优先检查是否需要关闭当前 Tab
    autocmd BufEnter * call <SID>CloseLonelyNERDTree()

    " 新 Tab 首次进入代码 Buffer 时初始化目录树
    autocmd BufEnter * call <SID>InitNERDTree()

augroup END


