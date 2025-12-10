## 异常处理指南

### DICOM 服务器代码遵循以下模式来引发异常
- DICOM 服务器代码中抛出的所有异常都继承自基类型 DicomServerException。
- 所有用户输入验证错误都抛出派生自 ValidationException 的异常。
- 内部类使用 Ensure 库来验证输入。Ensure 库抛出 .Net Argument*Exception。
- 来自依赖库（如 fo-dicom）的异常被捕获并包装在继承自 DicomServerException 的异常中。
- 来自依赖服务库（如 Azure 存储 blob）的异常被捕获并包装在继承自 DicomServerException 的异常中。

### DICOM 服务器代码遵循以下模式来处理异常
- 所有 DicomServerExceptions 都在中间件 [ExceptionHandlingMiddleware](/src/Microsoft.Health.Dicom.Api/Features/Exceptions/ExceptionHandlingMiddleware.cs) 中处理。这些异常被映射到正确的状态代码和响应体。
- 所有意外异常都被记录并映射到 500 服务器错误。
