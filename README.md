<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>إضافة خيار اجتماع</title>

    <!-- Bootstrap -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet">

    <!-- jQuery -->
    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>

    <style>
        fieldset {
            padding: 20px;
            border: 2px solid #29504d;
            border-radius: 10px;
            background-color: #ffffff;
            box-shadow: 0 3px 10px rgba(0, 0, 0, 0.1);
            margin-top: 5px;
        }

        legend {
            font-size: 18px;
            padding: 0 10px;
            color: #29504d !important;
        }

        label {
            font-size: 15px;
            color: #333;
            margin-bottom: 5px;
        }
    </style>
</head>

<body class="p-4">

    <!-- رسالة النجاح -->
    <div id="successMessage" class="alert alert-success text-center fw-bold"
         style="display:none; border-radius:10px;">
        تم حفظ الخيار بنجاح
    </div>

    <!-- اسم الخيار -->
    <fieldset>
        <legend class="fw-bold">اسم الخيار</legend>

        <div class="mb-3">
            <label>اسم الخيار</label>
            <input type="text" id="optionName" name="OptionName" class="form-control" placeholder="مثال: اعتماد النظام الجديد" required />
            <span class="text-danger" id="optionNameError"></span>
        </div>
    </fieldset>

    <!-- الإيجابيات -->
    <fieldset>
        <legend class="fw-bold">الإيجابيات</legend>

        <div class="table-responsive">
            <table class="table">
                <thead>
                    <tr>
                        <th>الإيجابية</th>
                        <th>إجراءات</th>
                    </tr>
                </thead>
                <tbody id="prosTable"></tbody>
            </table>

            <button type="button" class="btn btn-success mb-1"
                    style="border-radius: 14px; width:250px;"
                    id="addProRow">
                + إضافة إيجابية
            </button>
            <br />
            <span class="text-danger" id="prosError"></span>
        </div>
    </fieldset>

    <!-- السلبيات -->
    <fieldset>
        <legend class="fw-bold">السلبيات</legend>

        <div class="table-responsive">
            <table class="table">
                <thead>
                    <tr>
                        <th>السلبية</th>
                        <th>إجراءات</th>
                    </tr>
                </thead>
                <tbody id="consTable"></tbody>
            </table>

            <button type="button" class="btn btn-danger mb-1"
                    style="border-radius: 14px; width:250px;"
                    id="addConRow">
                + إضافة سلبية
            </button>
            <br />
            <span class="text-danger" id="consError"></span>
        </div>
    </fieldset>

    <!-- زر الحفظ -->
    <fieldset>
        <legend class="fw-bold">حفظ السجل</legend>

        <button type="button" id="saveOption" class="btn btn-primary"
                style="border-radius: 14px; width:250px; font-size:18px;">
            حفظ الخيار
        </button>
    </fieldset>

    <!-- جدول جميع الخيارات -->
    <fieldset>
        <legend class="fw-bold">جميع الخيارات</legend>

        <table class="table table-bordered" id="optionsTable">
            <thead>
                <tr>
                    <th>اسم الخيار</th>
                    <th>عدد الإيجابيات</th>
                    <th>عدد السلبيات</th>
                    <th>إجراءات</th>
                </tr>
            </thead>
            <tbody>
                <!-- الصفوف ستضاف هنا -->
            </tbody>
        </table>
    </fieldset>

    <script>
        let rowIndexPros = 0;
        let rowIndexCons = 0;

        // إضافة إيجابية
        $("#addProRow").on("click", function () {
            $("#prosTable").append(`
                <tr>
                    <td><input type="text" class="form-control pro-input" placeholder="اكتب إيجابية" required></td>
                    <td><button class="btn btn-danger remove-row">حذف</button></td>
                </tr>
            `);
        });

        // إضافة سلبية
        $("#addConRow").on("click", function () {
            $("#consTable").append(`
                <tr>
                    <td><input type="text" class="form-control con-input" placeholder="اكتب سلبية" required></td>
                    <td><button class="btn btn-danger remove-row">حذف</button></td>
                </tr>
            `);
        });

        // حذف صف
        $(document).on("click", ".remove-row", function () {
            $(this).closest("tr").remove();
        });

        // حفظ البيانات باستخدام AJAX
        $("#saveOption").on("click", function () {

            let isValid = true;

            $("#optionNameError").text("");
            $("#prosError").text("");
            $("#consError").text("");

            if ($("#optionName").val().trim() === "") {
                $("#optionNameError").text("الرجاء إدخال اسم الخيار");
                isValid = false;
            }

            if ($(".pro-input").length === 0) {
                $("#prosError").text("الرجاء إضافة إيجابية واحدة على الأقل");
                isValid = false;
            }

            if ($(".con-input").length === 0) {
                $("#consError").text("الرجاء إضافة سلبية واحدة على الأقل");
                isValid = false;
            }

            if (!isValid) return;

            let optionName = $("#optionName").val();

            let pros = [];
            $(".pro-input").each(function () {
                pros.push($(this).val());
            });

            let cons = [];
            $(".con-input").each(function () {
                cons.push($(this).val());
            });

            // إرسال البيانات
            $.ajax({
                url: "/Meeting/SaveOption",
                type: "POST",
                contentType: "application/json",
				data: JSON.stringify({
    OptionName: optionName,     
    Pros: pros,
    Cons: cons
}),

                success: function () {

                    $("#successMessage").text("تم حفظ الخيار بنجاح").fadeIn();
                    setTimeout(() => $("#successMessage").fadeOut(), 4000);

                    // إضافة الصف للجدول أسفل الشاشة
                    $("#optionsTable tbody").append(`
                        <tr>
                            <td class="opt-name">${optionName}</td>
                            <td class="opt-pros">${pros.length}</td>
                            <td class="opt-cons">${cons.length}</td>
                            <td>
                                <button class="btn btn-warning btn-sm edit-option">تعديل</button>
                                <button class="btn btn-danger btn-sm delete-option">حذف</button>
                            </td>
                        </tr>
                    `);

                    // تفريغ الحقول بعد الحفظ
                    $("#optionName").val("");
                    $("#prosTable").empty();
                    $("#consTable").empty();
                }
            });
        });

        // حذف خيار من الجدول
        $(document).on("click", ".delete-option", function () {
            $(this).closest("tr").remove();
        });

        // تعديل خيار
        $(document).on("click", ".edit-option", function () {

            let row = $(this).closest("tr");

            let name = row.find(".opt-name").text();
            let prosCount = parseInt(row.find(".opt-pros").text());
            let consCount = parseInt(row.find(".opt-cons").text());

            $("#optionName").val(name);

            $("#prosTable").empty();
            $("#consTable").empty();

            for (let i = 0; i < prosCount; i++) {
                $("#prosTable").append(`
                    <tr>
                        <td><input type="text" class="form-control pro-input" placeholder="اكتب إيجابية" required></td>
                        <td><button class="btn btn-danger remove-row">حذف</button></td>
                    </tr>
                `);
            }

            for (let i = 0; i < consCount; i++) {
                $("#consTable").append(`
                    <tr>
                        <td><input type="text" class="form-control con-input" placeholder="اكتب سلبية" required></td>
                        <td><button class="btn btn-danger remove-row">حذف</button></td>
                    </tr>
                `);
            }

            row.remove();

            $("#successMessage").text("تم تحميل البيانات للتعديل").fadeIn();
            setTimeout(() => $("#successMessage").fadeOut(), 3000);
        });

    </script>

</body>
</html>
