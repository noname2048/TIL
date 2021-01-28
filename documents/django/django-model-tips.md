## django-model-tips
admin 등록과정에서 list_display에 내용을 추가하기
```python
class Post(models.Model):
    # ...
    def message_length(self):
        return len(self.message)

    message_length.short_description = '메세지 글자수'

```
라고하면 admin에서 list_display에 message_length 등록 가능 (단, 함수에 인자가 없어야 한다.)
