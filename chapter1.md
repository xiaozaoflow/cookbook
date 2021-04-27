# First Chapter

123123112312312312

```java
    @Override
    public List<Picbook> pageByPicbookIds(List<Long> picbookIds, Long tagId, IPage<Picbook> page) {
        Long size = page.getSize();
        Long current = page.getCurrent();
        if (CommonConstants.DEFAULT_ZERO_ID >= current) {
            current = CommonConstant.ONE.longValue();
        }
        Long sqlCurrent = (current - CommonConstant.ONE) * size;
        return this.baseMapper.pageByPicbookIds(picbookIds, tagId, sqlCurrent, size);
    }
```

阿斯达

