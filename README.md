若依改用达梦数据库踩坑

1. 数据库从Mysql迁移到DM时候  数据库表名一定要是大写  这样查询的时候可以不用加引号

2. sys_menu

   Sys_MenuMapper.xml  中query的引号去掉

3. cast 转换取消  16进制须手动转换成字符串

4. 在 `MySQL` 中使用的函数 `find_in_set` 迁移到`DM8`后报错。

在DM数据库模式中执行

```sql
CREATE OR REPLACE FUNCTION FIND_IN_SET
                (
                        piv_str1 varchar2,
                        piv_str2 varchar2,
                        p_sep    varchar2 := ',')
                RETURN NUMBER
                            IS
                l_idx     number:=0;                 -- 用于计算piv_str2中分隔符的位置
                str       varchar2(500);             -- 根据分隔符截取的子字符串
                piv_str   varchar2(500) := piv_str2; -- 将piv_str2赋值给piv_str
                res       number        :=0;         -- 返回结果
                loopIndex number        :=0;
        BEGIN
                -- 如果piv_str中没有分割符，直接判断piv_str1和piv_str是否相等，相等 res=1
                IF instr(piv_str, p_sep, 1) = 0 THEN
                        IF piv_str          = piv_str1 THEN
                                res        := 1;
                        END IF;
                ELSE
                        -- 循环按分隔符截取piv_str
                        LOOP
                                l_idx    := instr(piv_str, p_sep);
                                loopIndex:=loopIndex+1;
                                -- 当piv_str中还有分隔符时
                                IF l_idx > 0 THEN
                                        -- 截取第一个分隔符前的字段str
                                        str:= substr(piv_str, 1, l_idx-1);
                                        -- 判断 str 和piv_str1 是否相等，相等 res=1 并结束循环判断
                                        IF str      = piv_str1 THEN
                                                res:= loopIndex;
                                                EXIT;
                                        END IF;
                                        piv_str := substr(piv_str, l_idx+length(p_sep));
                                ELSE
                                        -- 当截取后的piv_str 中不存在分割符时，判断piv_str和piv_str1是否相等，相等 res=1
                                        IF piv_str  = piv_str1 THEN
                                                res:= loopIndex;
                                        END IF;
                                        -- 无论最后是否相等，都跳出循环
                                        EXIT;
                                END IF;
                        END LOOP;
                        -- 结束循环
                END IF;
                -- 返回res
                RETURN res;
        END FIND_IN_SET;
commit;
```



- user为达梦数据库关键字，不能用作表名称
- 达梦数据库不支持“`”
- 达梦数据库返回得字段名全部转化为大写
- 达梦数据库字段类型转换 cast( 字段名 as 类型)  例：cast(dept_id as varchar)
- Text类型要转换为字符串类型
- mysql中group_concat 使用 wm_concat替换
- 达梦数据库在 select 需要查询的语句中选中的字段，必须出现在 GROUP BY 子句中
- FIND_IN_SET函数需要自定义
- 表名、字段名用”括起来
- 查询语句中的值类型用’括起来，不能用"
- 达梦limit最大值9223372036854775807
- if(true,2,3)  第二和第三个参数必须是int类型
- 使用DBMS_LOB.SUBSTR函数转换clob类型的数据到字符串
