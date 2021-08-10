# OneShot: Integer Overflow 

Integer overflow to overwrite the variable that is being checked (located at 0x404068)

```python
hex(0xffffffffffffffff - 0x500000 + 0x404068 + 1)
```