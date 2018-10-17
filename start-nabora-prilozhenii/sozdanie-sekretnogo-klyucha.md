### Создание секретного ключа

Сгенерируйте случайную строку, которая будет выступать в качестве секретного ключа, следующей командой:

```
$ python -c "import string, random; uni=string.ascii_letters+string.digits+string.punctuation; print repr(''.join([random.SystemRandom().choice(uni) for i in range(random.randint(45,50))]))"
```



