# Custom Validation

## Custom attributes

Custom attributes inherit from `ValidationAttribute`, which declares a method `IsValid` to validate an object.

```c#
public class AgeValidator : ValidationAttribute
{
  public int MinAge { get; set; }

  public override bool IsValid(object value) {
    DateTime dob;
    if (value != null &&
      DateTime.TryParse(value.ToString(), out dob)) {
        if (dob.AddYears(MinAge)  <= DateTime.Now) {
          return true;
        }
    }
    return false;
  }
}
```

```c#
// on model
public class Applicant
{
  [AgeValidator(Age = 18)]
  public string Dob { get; set; }
}
```

## Mutual validation

The value of some fields will indicate the validation of other fields.

For example,say a form requires a job applicant's length of time with their current employer. If the length is less than 2 years, it requires information on their previous employer. Validating the additional data depends on the value of their current job length.

This behaviour is not possible with attributes, instead the model class inherits `IValidateObject` which uses the `Validate` method to return a list of `ValidationResult`

```c#
public class Application
{
  // general info etc
  [Required]
  [MaxLength(60)]
  public string FirstName { get; set; }
  public int CurrentEmploymentLength { get; set; }

  // previous info
  public string PreviousRole { get; set; }
  public string PreviousEmployerName { get; set; }
  public int PreviousLength { get; set; }

  public IEnumerable<ValidationResult> Validate(ValidationContext context) {
    var errors = new List<ValidationResult>();

    if (CurrentEmploymentLength < 2) {
      // check validation of previous fields
    }

    return errors;
  }
}
```
