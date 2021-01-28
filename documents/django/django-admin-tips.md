## django-admin-tips
tips
### admin 등록 과정에서
데코레이터(장식자)를 이용할 수 있다.
```python
@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['id', 'message', 'created_at', 'updated_at']
    list_display_links = ['message']
```
이때 admin 에 나오는 링크를 message 로 설정가능
```python
# 원래는
admin.site.register(Post)
# 혹은
admin.site.register(Post, PostAdmin)
```
admin에 display_list를 구현하고 싶다면 (admin 에서만 쓸거면)
