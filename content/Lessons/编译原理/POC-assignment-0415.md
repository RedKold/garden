- `a = b[i] + c[j]`

```
			op		arg1	arg2	result
x=y[i]:		=[]		y		i		x
x[i]=y:		[]=		i		y		x

			__		op		arg1	arg2
x=y[i]:		0		=[]		y		i
			1		=		x		(0)			
x[i]=y:
			0		+		x		i
			1		[]=		(0)		y
```


```
			op		arg1	arg2	result
			[]		b		i		t1
			[]		c		j		t2
			+		t1		t2		a
```

```
			op		arg1	arg2	result
			*
```