# Apex

## Execute Anonymous Apex

```bash
# Inline (opens stdin prompt)
sf apex run --target-org my-org

# From file
sf apex run --file scripts/my-script.apex --target-org my-org
```

## Run Apex Tests

```bash
# Run all tests
sf apex run test --target-org my-org --wait 10

# Run specific test classes
sf apex run test --class-names MyTestClass --target-org my-org --wait 10
sf apex run test --class-names "MyTestClass,AnotherTest" --target-org my-org

# Run specific test methods
sf apex run test --tests MyTestClass.testMethod1 --target-org my-org

# Run tests in a suite
sf apex run test --suite-names MySuite --target-org my-org

# Code coverage
sf apex run test --code-coverage --target-org my-org --wait 10

# Async (get results later)
sf apex run test --class-names MyTestClass --target-org my-org
sf apex get test --test-run-id 707... --target-org my-org
```

## Apex Logs

```bash
sf apex list log --target-org my-org
sf apex get log --log-id <id> --target-org my-org
sf apex get log --number 5 --target-org my-org          # Last 5 logs
sf apex get log --output-dir logs/ --number 10           # Save to directory
sf apex tail log --target-org my-org                     # Live tail
```

## Generate Apex Templates

```bash
sf apex generate class --name MyClass --output-dir force-app/main/default/classes
sf apex generate trigger --name MyTrigger --sobject Account --event "before insert,after insert"
```
