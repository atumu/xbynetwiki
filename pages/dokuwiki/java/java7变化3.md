title: java7变化3 

#  java7变化之简便地实现hashCode,equals方法 
使用Objects类的equals方法,Objects类的hash方法。例如:
return Objects.equals(first,o.first) && Objects.equals(last,o.last).它会自动检查null而不用再手动进行判断了。

return Objects,hash(first,last).可以接受多个参数，然后合并hash运算结果。