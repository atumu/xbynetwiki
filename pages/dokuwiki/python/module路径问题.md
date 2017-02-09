title: module路径问题 

#  module路径问题 
```

import sys,os
#print(os.path.dirname(__file__))
#print(os.path.abspath(os.path.dirname(__file__)))
#print(os.path.abspath(os.path.join(os.path.dirname(__file__),'..','..')))
sys.path.append(os.path.abspath(os.path.join(os.path.dirname(__file__),'..','..')))
#print(sys.path)

```