---
title: "ModelValidator驗證(service、repository)"
date: 2022-10-20T17:02:03+08:00
draft: false
tags: ["Csharp"]
type: post
author: "Steve Lin"
description: ""
slug: "ModelValidator驗證"
---

#### 可以在Service層或Repository層做參數驗證
- 會使用到System.ComponentModel.DataAnnotations
- 一樣在參數上掛驗證Attribute
```C#
 public class InputParameter
    {
        [Required]
        public int[] arr1 { get; set; }

        [Required]
        public int[] arr2 { get; set; }

        [StringLength(5)]
        public string IP { get; set; }
    }
```

### 驗證方法
```C#
public class ModelValidator
{
    /// <summary>
    /// Validates the specified model.
    /// </summary>
    /// <typeparam name="T"></typeparam>
    /// <param name="model">The model.</param>
    /// <param name="parameterName">parameterName</param>
    /// <exception cref="ArgumentException"></exception>
    public static void Validate<T>(T model, string parameterName) where T : class
    {
        if (model is null)
        {
            if (string.IsNullOrWhiteSpace(parameterName))
            {
                parameterName = nameof(model);
            }
            throw new ArgumentNullException(paramName: parameterName, message: $"The value '{parameterName}' cannot be null.");
        }

        var errors = new List<ValidationResult>();
        var validation = Validator.TryValidateObject(model, new ValidationContext(model), errors, true);
        if (validation.Equals(false) && errors.Any().Equals(true))
        {
            var error = errors.FirstOrDefault();
            throw new ArgumentException(message: error.ErrorMessage, paramName: error.MemberNames.FirstOrDefault());
        }
    }
}
```
