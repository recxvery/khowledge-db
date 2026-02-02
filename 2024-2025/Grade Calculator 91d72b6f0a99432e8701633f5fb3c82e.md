# Grade Calculator

<aside>
💡 **Notion Tip:** Calculate your grade in any class without the legwork.

- Your `Raw Score` is the grade you got without a late penalty.
- The `Final Grade` calculates your late penalty. You can hide the late penalty column after you make sure the amount is correct.
- If your late assignment was excused, check off the `Excused` property and your late penalty will be ignored.
- The sum of grades beneath the `Weighted Grade` column is your grade in the class.
- How the template works 🔎
    
    ## **Days Late**
    
    This formula calculates the number of days between the due date and submission.
    In case of excused tardiness, correct the submission date or delete the formula and turn this column into a simple number.
    
    ```notion
    dateBetween(prop("Due"), prop("Submitted"), "days") * -1
    ```
    
    ## **Final Grade**
    
    This formula takes late policies into consideration. It also accounts for any tardiness that's been excused.
    
    ```notion
    prop("Grade") - prop("Days Late") * prop("Late Penalty") * 100 * toNumber(not prop("Excused"))
    ```
    
    ## **Class Grade**
    
    This formula calculates points per assignment. The sum is your grade in the class.
    
    ```notion
    prop("Final Grade") * prop("Weighting")
    ```
    
    **Note**: If you modify any of the table's properties, you'll need to adjust the above formulas accordingly. 
    
</aside>

↓ Click the button to add a new table for another course

[English 001](Grade%20Calculator/English%20001%2006e1422eb1954fbdabe5e22c302510d3.csv)