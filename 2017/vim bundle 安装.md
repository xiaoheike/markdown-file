## ��װ vundle ##
`vundle` ������ `Git`���������ȵ�Ҫ�� `Git` ��װ��ɺ����û����������ӹ��� `Clone Vundle` ��·�� `~/.vim/bundle/Vundle.vim` �£�`~` ��ָ `home` Ŀ¼���ҵ� `home` ·��Ϊ `C:\Users\xiaoheike\`�����鲻���޸ģ��޸��˷����鷳��
```json
$ git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

## ����vundle ##
�� `_vimrc` �ļ��м������������

"=====Vundle ���������=====
set nocompatible                                "���� Vi ����ģʽ  
filetype off                                    "�����ļ��������  

" �����װ��·���ı��ˣ��������·��ҲҪ�ı�
set rtp+=set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()

" ���鲻Ҫ�������·�������ƥ�䣬·��Ҳ��Ҫ������ vim �� plugins Ŀ¼��bundle ���޷�����ƥ��Ĳ����
" call vundle#begin('~/some/path/here')
   
" ʹ��Vundle�����������������Ҫ�С�  
Plugin 'VundleVim/Vundle.vim' 
   
" ����ΪҪ��װ����µĲ��

" ���еĲ���������֮ǰƥ��
call vundle#end()
filetype plugin indent on


��������
Vundle����ָ��
:PluginList �г��Ѿ���װ�Ĳ��
:PluginInstall ��װ���������ļ��еĲ��
:PluginInstall ��װ���������ļ��еĲ��
:PluginInstall! �������в��
:PluginSearch pluginName �������
:PluginClean! ���������ļ�ɾ�����

 
