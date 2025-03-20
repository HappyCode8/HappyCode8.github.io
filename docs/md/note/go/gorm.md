1. update时，如果没更新数据要报错，result.RowsAffected == 0 加一个业务错误码
   
   ```go
   //一个错误码
   if err != nil {
      bizErr = errdef.NewBizErr(errdef.MysqlException, err, "")
      return
   }
   //另一个错误码
   if result.RowsAffected == 0 {
      bizErr = errdef.NewParamsErr("no column update")
      return nil
   }
   ```

2. 如果没查出数据，是报错还是返回
   
   - 对于take、first、Last，**如果记录不存在会报错 record not found**
   
   - 对于scan方法，**如果记录不存在不会报错，但是的到的数据是一个 "零值"**
   
   - 对于Find方法，***如果记录不存在不会报错，但是会返回一个"空结构体",里面的值是结构体属性的"零值"**
   
   结论：
   
   > 如果要报错，单独判断errors.Is(err, gorm.ErrRecordNotFound)，然后加一个单独的错误码，方法名最好命名为mustGetOneXXXX这样子
   > 
   > 如果不报错，单独判断errors.Is(err, gorm.ErrRecordNotFound)吞掉，然后返回nil，由上层方法处理

3. 关于更新没有正常更新数据，是报错还是返回
   
   - update的每一个字段，最好新建一个model同时包括更新条件和更新数据，然后全部设为指针类型，根据赋值情况进行更新，方法一般命名为updateByConditon
   
   结论：
   
   > 如果要报错，根据result.RowsAffected判断，如果没更新的需要明确判断，单独判断这一个结果，然后加一个单独的错误码，方法名最好命名为mustUpdateXXXX这样子
   > 
   > 如果不报错，不处理就行
   > 
   > 更进一步地，可以在model里加入一个参数，决定是吞掉还是报错

4. 如果是取多个数据，Mget命名，返回最好同时有list和map

5. 如果有分页查询，在查询的model condition里，加一个是否要by page查询

6. service时一次查出来数据(可以并行)，然后处理业务，然后打包，三部流程
