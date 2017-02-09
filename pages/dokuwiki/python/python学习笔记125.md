title: python学习笔记125 

#  Python总结之算法例子 
		斐波那契
			#将函数结果作为列表可用于循环
			def fab(max): 
			n, a, b = 0, 0, 1 
			while n < max: 
				yield b         
				a, b = b, a + b 
				n = n + 1 
			for n in fab(5): 
				print n

		乘法口诀

			#!/usr/bin/python
			for i in range(1,10):
				for j in range(1,i+1):
					print j,'*',i,'=',j*i,
				else:
					print ''

		最小公倍数

			# 1-70的最小公倍数
			def c(m,n):
					a1=m
					b1=n
					r=n%m
					while r!=0:
							n=m
							m=r
							r=n%m
					return (a1*b1)/m
			d=1
			for i in range(3,71,2):
					d = c(d,i)
			print d

		排序算法

			插入排序
				def insertion_sort(sort_list):
					iter_len = len(sort_list)
					if iter_len < 2:
						return sort_list
					for i in range(1, iter_len):
						key = sort_list[i]
						j = i - 1
						while j>=0 and sort_list[j]>key:
							sort_list[j+1] = sort_list[j]
							j -= 1
						sort_list[j+1] = key
					return sort_list

			选择排序
				def selection_sort(sort_list):
					iter_len = len(sort_list)
					if iter_len < 2:
						return sort_list
					for i in range(iter_len-1):
						smallest = sort_list[i]
						location = i
						for j in range(i, iter_len):
							if sort_list[j] < smallest:
								smallest = sort_list[j]
								location = j
						if i != location:
							sort_list[i], sort_list[location] = sort_list[location], sort_list[i]
					return sort_list	

			冒泡排序算法
				def bubblesort(numbers):
					for j in range(len(numbers)-1,-1,-1):
						for i in range(j):
							if numbers[i]>numbers[i+1]:
								numbers[i],numbers[i+1] = numbers[i+1],numbers[i]
							print(i,j)
							print(numbers)

		二分算法

			#python 2f.py 123456789 4
			# list('123456789')  =  ['1', '2', '3', '4', '5', '6', '7', '8', '9']
			#!/usr/bin/env python 
			import sys

			def search2(a,m):
				low = 0
				high = len(a) - 1
				while(low <= high):
					mid = (low + high)/2
					midval = a[mid]

					if midval < m:
						low = mid + 1
					elif midval > m:
						high = mid - 1
					else:
						print mid
						return mid
				print -1
				return -1

			if __name__ == "__main__":
				a = [int(i) for i in list(sys.argv[1])]
				m = int(sys.argv[2])
				search2(a,m)
		
	将字典中所有time去掉
	
		a={'version01': {'nba': {'timenba': 'valuesasdfasdf', 'nbanbac': 'vtimefasdf', 'userasdf': 'vtimasdf'}}}
		eval(str(a).replace("time",""))