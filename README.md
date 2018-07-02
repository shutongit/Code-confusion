> 最近在做银行的项目，所以对安全性要求很高。这不安全检测没过??，这里面的问题就提到了代码混淆问题
####准备工作
 1. cd到你自己的项目目录级![这里写图片描述](https://img-blog.csdn.net/20180702155621252?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NodVRvbmdJdA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 2. 创建confuse.sh文件和func.list文件![这里写图片描述](https://img-blog.csdn.net/20180702155929718?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NodVRvbmdJdA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 3. 选中项目选择运行脚本。这里需要注意的是`$PROJECT_DIR/confuse.sh`这个的路径和创建pch文件时的路径是一样的，`$PROJECT_DIR`代表整个工程，`/confuse.sh`是这个文件的路径。 ![这里写图片描述](https://img-blog.csdn.net/20180702160435777?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NodVRvbmdJdA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 4. 然后command + B编译一下，如果报错了就cd到项目的目录级下，然后输入命令行 `chmod 755 confuse.sh`或`chmod 777 confuse.sh` 给我们的脚本本间授权
 5. 成功之后会自动生成一个`codeObfuscation.h`文件,注意：如果没有生成也可以自己创建一个空白的.h文件![这里写图片描述](https://img-blog.csdn.net/20180702161020922?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NodVRvbmdJdA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 6. 然后在pch文件中导入`codeObfuscation.h`文件。![这里写图片描述](https://img-blog.csdn.net/20180702161206535?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NodVRvbmdJdA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

####重点来了

 1. 全局混淆。如果我们之前没有考虑到混淆的问题现在用全局混淆并不明智，因为有的时候方法名的命名并不是那么规范，这里我们全局混淆以`sk_`开头的方法名，如果不想做特定的限制的话可以把`|sed -n "/^sk_/p"`这里给删除掉，我们来看下效果：![这里写图片描述](https://img-blog.csdn.net/20180702163500285?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NodVRvbmdJdA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)![这里写图片描述](https://img-blog.csdn.net/20180702163526757?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NodVRvbmdJdA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)![这里写图片描述](https://img-blog.csdn.net/20180702163739109?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NodVRvbmdJdA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)![这里写图片描述](https://img-blog.csdn.net/20180702163536852?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NodVRvbmdJdA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 2. 局部混淆。可以在`func.list`文件中手动添加方法。编译之后就可以了![这里写图片描述](https://img-blog.csdn.net/20180702164420407?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1NodVRvbmdJdA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 
 
 ####重中之重

```
#!/usr/bin/env bash

TABLENAME=symbols
SYMBOL_DB_FILE="symbols"
STRING_SYMBOL_FILE="func.list"
HEAD_FILE="$PROJECT_DIR/$PROJECT_NAME/codeObfuscation.h"
####这里是全局混淆要查找的内容路径，不需要全局混淆的时候要注释掉
CONFUSE_FILE="$PROJECT_DIR/代码混淆"
export LC_CTYPE=C
####这里是全局混淆要查找的条件，不需要全局混淆的时候要注释掉
#取以.m或.h结尾的文件以+号或-号开头的行 |去掉所有+号或－号|用空格代替符号|n个空格跟着<号 替换成 <号|开头不能是IBAction|用空格split字串取第二部分|排序|去重复|删除空行|删掉以init开头的行>写进func.list
## |sed -n "/^sk_/p" 是特定问方法名的开头，不需要的话可以删掉
grep -h -r -I  "^[-+]" $CONFUSE_FILE  --include '*.[mh]' |sed "s/[+-]//g"|sed "s/[();,: *\^\/\{]/ /g"|sed "s/[ ]*</</"| sed "/^[ ]*IBAction/d"|awk '{split($0,b," "); print b[2]; }'| sort|uniq |sed "/^$/d" |sed -n "/^sk_/p" >$STRING_SYMBOL_FILE

#维护数据库方便日后作排重
createTable()
{
echo "create table $TABLENAME(src text, des text);" | sqlite3 $SYMBOL_DB_FILE
}

insertValue()
{
echo "insert into $TABLENAME values('$1' ,'$2');" | sqlite3 $SYMBOL_DB_FILE
}

query()
{
echo "select * from $TABLENAME where src='$1';" | sqlite3 $SYMBOL_DB_FILE
}

ramdomString()
{
openssl rand -base64 64 | tr -cd 'a-zA-Z' |head -c 16
}

rm -f $SYMBOL_DB_FILE
rm -f $HEAD_FILE
createTable

touch $HEAD_FILE
echo '#ifndef Demo_codeObfuscation_h
#define Demo_codeObfuscation_h' >> $HEAD_FILE
echo "//confuse string at `date`" >> $HEAD_FILE
cat "$STRING_SYMBOL_FILE" | while read -ra line; do
if [[ ! -z "$line" ]]; then
ramdom=`ramdomString`
echo $line $ramdom
insertValue $line $ramdom
echo "#define $line $ramdom" >> $HEAD_FILE
fi
done
echo "#endif" >> $HEAD_FILE


sqlite3 $SYMBOL_DB_FILE .dump

```
