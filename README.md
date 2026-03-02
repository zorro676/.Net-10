@model List<Webmeetingapp.ViewModel.OptionViewModel>

@{
    Layout = "~/Views/Shared/_Layout.cshtml";
}

<style>
    fieldset {
        border-radius: 6px;
        background: #fafafa;
        margin-bottom: 20px;
        padding: 15px;
    }

    legend {
        font-weight: bold;
        font-size: 18px;
    }

    .text-danger {
        font-size: 14px;
        margin-top: 5px;
        display: block;
    }
</style>

<div class="p-4">

    <input type="hidden" id="meetingId" value="122" />

    <div id="successMessage" class="alert alert-success text-center fw-bold" style="display:none;">
        تمت العملية بنجاح
    </div>

    <!-- اسم الخيار -->
    <fieldset>
        <legend>اسم الخيار</legend>
        <input type="text" id="optionName" class="form-control" placeholder="اسم الخيار">
        <span id="optionNameError" class="text-danger"></span>
    </fieldset>

    <!-- الإيجابيات -->
    <fieldset>
        <legend>الإيجابيات</legend>
        <table class="table table-bordered">
            <tbody id="prosTable"></tbody>
        </table>
        <button id="addProRow" class="btn btn-success">+ إضافة إيجابية</button>
        <span id="prosError" class="text-danger"></span>
    </fieldset>

    <!-- السلبيات -->
    <fieldset>
        <legend>السلبيات</legend>
        <table class="table table-bordered">
            <tbody id="consTable"></tbody>
        </table>
        <button id="addConRow" class="btn btn-danger">+ إضافة سلبية</button>
        <span id="consError" class="text-danger"></span>
    </fieldset>

    <!-- زر الحفظ -->
    <fieldset>
        <legend>حفظ</legend>
        <button id="saveOption" class="btn btn-primary">حفظ الخيار</button>
    </fieldset>

    <!-- جدول جميع الخيارات -->
    <fieldset>
        <legend>جميع الخيارات</legend>
        <table class="table table-striped table-bordered" id="optionsTable">
            <thead class="table-dark">
                <tr>
                    <th>اسم الخيار</th>
                    <th>عدد الإيجابيات</th>
                    <th>عدد السلبيات</th>
                    <th>إجراءات</th>
                </tr>
            </thead>
            <tbody>
                @foreach (var item in Model)
                {
                    <tr data-id="@item.OptionId">
                        <td>@item.OptionName</td>
                        <td>@item.ProsCount</td>
                        <td>@item.ConsCount</td>
                        <td>
                            <button class="btn btn-info btn-sm" onclick="viewOption(@item.OptionId)">عرض</button>
                            <button class="btn btn-warning edit-option">تعديل</button>
                            <button class="btn btn-danger delete-option" data-id="@item.OptionId">حذف</button>

                        </td>
                    </tr>
                }
            </tbody>
        </table>
    </fieldset>
</div>
@*<a href="/Home/DecisionsToBeMade" class="btn btn-primary">
    التالي معلومات القرار
</a>*@
<a id="nextBtn" href="/Home/DecisionsToBeMade" class="btn btn-primary">
    التالي معلومات القرار
</a>


<!-- نافذة عرض وتعديل -->

<div class="modal fade" id="optionModal" tabindex="-1">
    <div class="modal-dialog modal-lg">
        <div class="modal-content">

            <div class="modal-header">
                <h5 class="modal-title">تفاصيل الخيار</h5>
                <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
            </div>

            <div class="modal-body">

                <input type="hidden" id="editOptionId" />

                <!-- اسم الخيار -->
                <div class="mb-3">
                    <label class="form-label">اسم الخيار</label>
                    <input type="text" id="editOptionName" class="form-control">
                    <div id="editOptionNameError" class="text-danger mt-1"></div>
                </div>

                <!-- الإيجابيات -->
                <h6>الإيجابيات</h6>
                <table class="table table-bordered">
                    <tbody id="editProsTable"></tbody>
                </table>
                <div id="editProsError" class="text-danger mt-1"></div>
                <button id="addEditProRow" class="btn btn-success">+ إضافة إيجابية</button>

                <hr>

                <!-- السلبيات -->
                <h6>السلبيات</h6>
                <table class="table table-bordered">
                    <tbody id="editConsTable"></tbody>
                </table>
                <div id="editConsError" class="text-danger mt-1"></div>
                <button id="addEditConRow" class="btn btn-danger">+ إضافة سلبية</button>

            </div>

            <div class="modal-footer">
                <button class="btn btn-primary" id="saveEditOption">حفظ التعديلات</button>
            </div>

        </div>
    </div>
</div>
<div class="offcanvas offcanvas-end" tabindex="-1" id="viewOptionCanvas">
    <div class="offcanvas-header">
        <h5 class="offcanvas-title">تفاصيل الخيار</h5>
        <button type="button" class="btn-close" data-bs-dismiss="offcanvas"></button>
    </div>

    <div class="offcanvas-body">

        <div class="mb-3">
            <label class="fw-bold">اسم الخيار:</label>
            <div id="viewOptionName" class="p-2 bg-light border rounded"></div>
        </div>

        <h6 class="fw-bold">الإيجابيات:</h6>
        <ul id="viewProsList" class="list-group mb-3"></ul>

        <h6 class="fw-bold">السلبيات:</h6>
        <ul id="viewConsList" class="list-group"></ul>

    </div>
</div>


@section Scripts {

    <script>

        function checkOptionsTable() {
            let rowCount = $("#optionsTable tbody tr").length;

            if (rowCount > 0) {
                $("#nextBtn").show();   // يوجد بيانات → إظهار الزر
            } else {
                $("#nextBtn").hide();   // لا يوجد بيانات → إخفاء الزر
            }
        }

        // تشغيل الفحص عند تحميل الصفحة
        $(document).ready(function () {
            checkOptionsTable();
        });



        // إضافة إيجابية
        $("#addProRow").click(function () {
            $("#prosTable").append(`
                                    <tr>
                                        <td><input type="text" class="form-control pro-input" placeholder="اكتب إيجابية"></td>
                                        <td><button class="btn btn-danger remove-row">حذف</button></td>
                                    </tr>
                                `);
        });

        // إضافة سلبية
        $("#addConRow").click(function () {
            $("#consTable").append(`
                                    <tr>
                                        <td><input type="text" class="form-control con-input" placeholder="اكتب سلبية"></td>
                                        <td><button class="btn btn-danger remove-row">حذف</button></td>
                                    </tr>
                                `);
        });

        // حذف صف
        $(document).on("click", ".remove-row", function () {
            $(this).closest("tr").remove();
        });

        // حفظ خيار جديد باستخدام OptionEditViewModel
        $("#saveOption").click(function () {

            // إعادة تعيين رسائل الخطأ
            $("#optionNameError").text("");
            $("#prosError").text("");
            $("#consError").text("");

            let isValid = true;

            // التحقق من اسم الخيار
            let optionName = $("#optionName").val().trim();
            if (optionName === "") {
                $("#optionNameError").text("الرجاء إدخال اسم الخيار");
                isValid = false;
            }

            // التحقق من الإيجابيات
            let pros = [];
            $(".pro-input").each(function () {
                let v = $(this).val().trim();
                if (v === "") {
                    $("#prosError").text("لا يمكن ترك الإيجابيات فارغة");
                    isValid = false;
                }
                pros.push(v);
            });

            if (pros.length === 0) {
                $("#prosError").text("الرجاء إضافة إيجابية واحدة على الأقل");
                isValid = false;
            }

            // التحقق من السلبيات
            let cons = [];
            $(".con-input").each(function () {
                let v = $(this).val().trim();
                if (v === "") {
                    $("#consError").text("لا يمكن ترك السلبيات فارغة");
                    isValid = false;
                }
                cons.push(v);
            });

            if (cons.length === 0) {
                $("#consError").text("الرجاء إضافة سلبية واحدة على الأقل");
                isValid = false;
            }

            // إيقاف التنفيذ إذا كان هناك خطأ
            if (!isValid) return;

            // بناء الموديل الصحيح
            let model = {
                Id: null,
                MeetingId: $("#meetingId").val(),
                OptionName: optionName,
                Pros: pros,
                Cons: cons
            };

            // منع النقر المزدوج
            $("#saveOption").prop("disabled", true);

            // إرسال البيانات
            $.ajax({
                url: "/Home/SaveOrUpdateOption",
                type: "POST",
                contentType: "application/json",
                data: JSON.stringify(model),
                success: function (res) {
                    if (res.success) {
                        location.reload();
                    } else {
                        alert("لم يتم الحفظ، تحقق من البيانات");
                        $("#saveOption").prop("disabled", false);
                    }
                },
                error: function () {
                    alert("حدث خطأ أثناء حفظ البيانات");
                    $("#saveOption").prop("disabled", false);
                }
            });
        });

        // عرض أو تعديل خيار
        $(document).on("click", ".view-option, .edit-option", function () {

            let id = $(this).closest("tr").data("id");

            $.get("/Home/GetOptionDetails?id=" + id)
                .done(function (res) {

                    if (!res.success) {
                        alert((res.message || "") + "\n" + (res.error || ""));
                        return;
                    }

                    let data = res.data;

                    // تعبئة الحقول
                    $("#editOptionId").val(data.optionId);
                    $("#editOptionName").val(data.optionName);

                    // تنظيف الجداول
                    $("#editProsTable").empty();
                    $("#editConsTable").empty();

                    // دالة لإضافة صف
                    function addRow(tableSelector, value, inputClass) {
                        value = $("<div>").text(value).html(); // حماية XSS

                        $(tableSelector).append(`
                                    <tr>
                                        <td><input type="text" class="form-control ${inputClass}" value="${value}"></td>
                                        <td><button class="btn btn-danger remove-row">حذف</button></td>
                                    </tr>
                                `);
                    }

                    // إضافة الإيجابيات
                    data.pros.forEach(p => addRow("#editProsTable", p, "edit-pro-input"));

                    // إضافة السلبيات
                    data.cons.forEach(c => addRow("#editConsTable", c, "edit-con-input"));

                    $("#optionModal").modal("show");
                })
                .fail(function () {
                    alert("حدث خطأ أثناء تحميل البيانات");
                });

        });



        // إضافة إيجابية داخل المودال
        $("#addEditProRow").click(function () {
            $("#editProsTable").append(`
                                    <tr>
                                        <td><input type="text" class="form-control edit-pro-input"></td>
                                        <td><button class="btn btn-danger remove-row">حذف</button></td>
                                    </tr>
                                `);
        });

        // إضافة سلبية داخل المودال
        $("#addEditConRow").click(function () {
            $("#editConsTable").append(`
                                    <tr>
                                        <td><input type="text" class="form-control edit-con-input"></td>
                                        <td><button class="btn btn-danger remove-row">حذف</button></td>
                                    </tr>
                                `);
        });

        // حفظ التعديلات
        $("#saveEditOption").click(function () {

            // تنظيف رسائل الأخطاء
            $("#editOptionNameError").text("");
            $("#editProsError").text("");
            $("#editConsError").text("");

            let isValid = true;

            // التحقق من اسم الخيار
            let optionName = $("#editOptionName").val().trim();
            if (optionName === "") {
                $("#editOptionNameError").text("الرجاء إدخال اسم الخيار");
                isValid = false;
            }

            // جمع الإيجابيات
            let pros = [];
            $(".edit-pro-input").each(function () {
                let v = $(this).val().trim();
                if (v === "") {
                    $("#editProsError").text("لا يمكن ترك الإيجابيات فارغة");
                    isValid = false;
                }
                pros.push(v);
            });

            if (pros.length === 0) {
                $("#editProsError").text("الرجاء إضافة إيجابية واحدة على الأقل");
                isValid = false;
            }

            // جمع السلبيات
            let cons = [];
            $(".edit-con-input").each(function () {
                let v = $(this).val().trim();
                if (v === "") {
                    $("#editConsError").text("لا يمكن ترك السلبيات فارغة");
                    isValid = false;
                }
                cons.push(v);
            });

            if (cons.length === 0) {
                $("#editConsError").text("الرجاء إضافة سلبية واحدة على الأقل");
                isValid = false;
            }

            if (!isValid) return;

            // بناء الموديل بشكل متوافق مع السيرفر
            let model = {
                id: $("#editOptionId").val() || null,
                meetingId: $("#meetingId").val(),
                optionName: optionName,
                pros: pros,
                cons: cons
            };

            // منع النقر المزدوج
            $("#saveEditOption").prop("disabled", true);

            $.ajax({
                url: "/Home/SaveOrUpdateOption",
                type: "POST",
                contentType: "application/json",
                data: JSON.stringify(model),
                success: function (res) {

                    if (!res.success) {
                        alert((res.message || "") + "\n" + (res.error || ""));
                        $("#saveEditOption").prop("disabled", false);
                        return;
                    }

                    location.reload();
                },
                error: function () {
                    alert("حدث خطأ أثناء حفظ البيانات");
                    $("#saveEditOption").prop("disabled", false);
                }
            });
        });

        function viewOption(id) {

            $.ajax({
                url: "/Home/GetOptionById",
                type: "GET",
                data: { id: id },
                success: function (res) {

                    if (!res || !res.success) {
                        alert(res && res.message ? res.message : "تعذر جلب بيانات الخيار");
                        return;
                    }

                    var option = res.data;

                    // تعبئة اسم الخيار
                    $("#viewOptionName").text(option.optionName);

                    // الإيجابيات
                    $("#viewProsList").empty();
                    option.pros.forEach(p => {
                        $("#viewProsList").append(`<li class="list-group-item">${p}</li>`);
                    });

                    // السلبيات
                    $("#viewConsList").empty();
                    option.cons.forEach(c => {
                        $("#viewConsList").append(`<li class="list-group-item">${c}</li>`);
                    });

                    // فتح السلايدر
                    new bootstrap.Offcanvas("#viewOptionCanvas").show();
                },
                error: function () {
                    alert("حدث خطأ أثناء الاتصال بالخادم");
                }
            });
        }

        $(document).on("click", ".delete-option", function () {
            let id = $(this).data("id");

            if (!confirm("هل أنت متأكد من حذف هذا الخيار؟")) return;

            $.ajax({
                url: "/Home/DeleteOption",
                type: "POST",
                data: { id: id },
                success: function (res) {
                    if (res.success) {
                        checkOptionsTable();
                        location.reload();
                    } else {
                        alert(res.message || "تعذر حذف الخيار");
                    }
                },
                error: function () {
                    alert("حدث خطأ أثناء الاتصال بالخادم");
                }
            });
        });


    </script>

}
----------------------------
@{
    Layout = "~/Views/Shared/_Layout.cshtml";
    ViewBag.Title = "إدارة المبررات";
}

<style>
    #JustificationsDesc {
        width: 100%; /* يجعل الحقل يأخذ كامل عرض البطاقة */
        min-height: 200px; /* ارتفاع أكبر */
        font-size: 18px; /* تكبير الخط */
        padding: 15px; /* مساحة داخلية مريحة */
        border-radius: 10px; /* حواف ناعمة */
    }

    /* عنوان الصفحة */
    h3 {
        font-weight: bold;
        color: #2c3e50;
        border-right: 5px solid #3498db;
        padding-right: 10px;
    }

    /* بطاقة الإدخال */
    .card {
        border-radius: 10px;
        border: 1px solid #dcdcdc;
    }

    .card-header {
        background: linear-gradient(to left, #3498db, #6dd5fa);
        color: white;
        font-weight: bold;
        border-radius: 10px 10px 0 0 !important;
    }

    /* أزرار جميلة */
    .styled-btn {
        padding: 10px 25px;
        font-size: 16px;
        border-radius: 8px;
        font-weight: 600;
        border: none;
        transition: 0.3s;
        box-shadow: 0 3px 8px rgba(0,0,0,0.15);
        background-color: #3498db;
        color: white;
    }

        .styled-btn:hover {
            background-color: #217dbb;
            transform: translateY(-2px);
        }

    /* زر حفظ وإرسال */
    .submit-final-btn {
        width: 100%;
        padding: 15px;
        font-size: 20px;
        font-weight: bold;
        background-color: #28a745;
        color: white;
        border: none;
        border-radius: 10px;
        margin-top: 20px;
        box-shadow: 0 4px 10px rgba(0,0,0,0.2);
        transition: 0.3s;
    }

        .submit-final-btn:hover {
            background-color: #218838;
            transform: translateY(-2px);
        }

    /* حاوية الأزرار */
    .btn-container {
        display: flex;
        gap: 10px;
        margin-top: 10px;
    }

    /* زر الرجوع العائم */
    .floating-back-btn {
        position: fixed;
        bottom: 25px;
        left: 25px;
        width: 55px;
        height: 55px;
        border-radius: 50%;
        background-color: #6c757d;
        color: white;
        border: none;
        font-size: 24px;
        font-weight: bold;
        box-shadow: 0 4px 10px rgba(0,0,0,0.3);
        cursor: pointer;
        transition: 0.3s;
        z-index: 9999;
    }

        .floating-back-btn:hover {
            background-color: #5a6268;
            transform: scale(1.1);
        }
</style>

<div class="container mt-4">

    <h3 class="mb-4">إدارة المبررات</h3>

    <div class="card mb-4">
        <div class="card-header">إضافة / تعديل مبرر</div>

        <div class="card-body">

            <input type="hidden" id="JustificationsId" />

            <div class="mb-3">
                <label class="form-label">نص المبرر</label>
                <textarea id="JustificationsDesc" class="form-control" rows="6"></textarea>
               

                <div id="justError" class="text-danger"></div>
            </div>

            <div class="mb-3">
                <label class="form-label">رقم الاجتماع</label>
                <input type="number" id="meetingId" class="form-control" />
            </div>

            <div class="btn-container">
                <button id="backButton" class="styled-btn">رجوع</button>
                <button id="saveJustification" class="styled-btn">حفظ</button>
            </div>

            

        </div>
    </div>

    <div id="JustificationsTable"></div>

</div>

<!-- زر الرجوع العائم -->
<button id="backFloatingBtn" class="floating-back-btn" title="رجوع">⬅</button>
<button id="submitFinal" class="submit-final-btn" style="display:none;">
    حفظ وإرسال
</button>

@section Scripts {

    <script>
  

        // زر الرجوع العائم
        $("#backFloatingBtn").click(function () {
            window.history.back();
        });

        // زر الرجوع العادي
        $("#backButton").click(function () {
            window.history.back();
        });

        // زر الحفظ العادي
        $("#saveJustification").click(function () {

            $("#justError").text("");

            let model = {
                JustificationsId: $("#JustificationsId").val(),
                JustificationsDesc: $("#JustificationsDesc").val().trim(),
                meetingId: $("#meetingId").val()
            };

            if (model.JustificationsDesc === "") {
                $("#justError").text("الرجاء إدخال نص المبرر");
                return;
            }

            $.ajax({
                url: "/Home/SaveJustification",
                type: "POST",
                contentType: "application/json",
                data: JSON.stringify(model),
                success: function (res) {
                    if (res.success) {
                        loadJustifications();

                        $("#JustificationsId").val("");
                        $("#JustificationsDesc").val("");
                        $("#meetingId").val("");
                    } else {
                        alert(res.message);
                    }
                }
            });
        });
        // زر التعديل
        $(document).on("click", ".edit-btn", function () {
            let id = $(this).data("id");

            $.get("/Home/GetJustificationById", { id: id }, function (j) {

                $("#JustificationsId").val(j.JustificationsId);
                $("#JustificationsDesc").val(j.JustificationsDesc);
                $("#meetingId").val(j.meetingId);

                // تمدد تلقائي بعد تعبئة النص
                autoExpand(document.getElementById("JustificationsDesc"));
            });
        });


        // زر الحذف
        // زر الحذف
        $(document).on("click", ".delete-btn", function () {

            let id = $(this).data("id");

            if (!confirm("هل أنت متأكد من حذف هذا المبرر؟")) return;

            $.post("/Home/DeleteJustification", { id: id }, function (res) {

                if (res.success) {

                    // إعادة تحميل البطاقات
                    loadJustifications();

                    // Reset لجميع الحقول
                    $("#JustificationsId").val("");
                    $("#JustificationsDesc").val("");
                    $("#meetingId").val("");

                    // إعادة ارتفاع حقل المبرر لوضعه الطبيعي
                    $("#JustificationsDesc").css("height", "auto");

                    // إخفاء زر حفظ وإرسال إذا لم يعد هناك بطاقات
                    $("#submitFinal").hide();

                } else {
                    alert(res.message);
                }
            });
        });


        // زر حفظ وإرسال (مستقل)
        $("#submitFinal").click(function () {

            let confirmSend = confirm("هل تريد حفظ وإرسال المبرر؟ يمكنك التعديل لاحقًا.");

            if (!confirmSend) return;

            let model = {
                JustificationsId: $("#JustificationsId").val(),
                JustificationsDesc: $("#JustificationsDesc").val().trim(),
                meetingId: $("#meetingId").val()
            };

            if (model.JustificationsDesc === "") {
                $("#justError").text("الرجاء إدخال نص المبرر");
                return;
            }

            $.ajax({
                url: "/Home/SubmitJustificationFinal",
                type: "POST",
                contentType: "application/json",
                data: JSON.stringify(model),
                success: function (res) {
                    if (res.success) {
                        alert("تم حفظ وإرسال المبرر بنجاح.");
                    } else {
                        alert(res.message);
                    }
                }
            });

        });

        // تحميل المبررات
        function loadJustifications() {
            $.get("/Home/GetAllJustifications", function (res) {

                $("#JustificationsTable").empty();

                // التحكم في ظهور زر حفظ وإرسال
                if (res.length === 0) {
                    $("#submitFinal").hide();   // إخفاء الزر إذا لا توجد بطاقات
                } else {
                    $("#submitFinal").show();   // إظهار الزر إذا توجد بطاقات
                }

                res.forEach(j => {

                    $("#JustificationsTable").append(`
                <div class="card mb-3 shadow-sm">

                    <div class="card-body">

                        <div class="row mb-2">
                            <div class="col-4 fw-bold text-end">المبرر:</div>
                            <div class="col-8">${j.JustificationsDesc}</div>
                        </div>

                        <div class="row mb-3">
                            <div class="col-4 fw-bold text-end">رقم الاجتماع:</div>
                            <div class="col-8">${j.meetingId}</div>
                        </div>

                        <div class="text-start">
                            <button class="btn btn-warning btn-sm edit-btn" data-id="${j.JustificationsId}">تعديل</button>
                            <button class="btn btn-danger btn-sm delete-btn" data-id="${j.JustificationsId}">حذف</button>
                        </div>

                    </div>
                </div>
            `);

                });
            });
        }


        // تحميل المبررات عند فتح الصفحة
        loadJustifications();

    </script>

}

--------------------------------

@{
    Layout = "~/Views/Shared/_Layout.cshtml";
    ViewBag.Title = "إدارة القرارات المطلوب اتخاذها";   // عنوان الصفحة
}
<style>

    /* تنسيق عنوان الصفحة */
    h3 {
        font-weight: bold;
        color: #2c3e50;
        border-right: 5px solid #3498db;
        padding-right: 10px;
    }

    /* تنسيق بطاقة الإدخال */
    .card {
        border-radius: 10px;
        border: 1px solid #dcdcdc;
    }

    .card-header {
        background: linear-gradient(to left, #3498db, #6dd5fa);
        color: white;
        font-weight: bold;
        border-radius: 10px 10px 0 0 !important;
    }

    /* تنسيق البطاقات المعروضة */
    .decision-card {
        transition: 0.3s;
        border-left: 5px solid #3498db;
    }

        .decision-card:hover {
            transform: scale(1.02);
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        }

    /* تنسيق العناوين داخل البطاقات */
    .label-title {
        font-weight: bold;
        color: #34495e;
    }

    /* زر التالي */
    #nextButton {
        font-size: 18px;
        padding: 10px 25px;
        border-radius: 8px;
    }


    /* زر الرجوع العائم */
    .floating-back-btn {
        position: fixed; /* يبقى ثابتًا عند التمرير */
        bottom: 25px; /* يبتعد 25px عن أسفل الشاشة */
        left: 25px; /* يبتعد 25px عن يسار الشاشة */
        width: 55px; /* عرض الزر */
        height: 55px; /* ارتفاع الزر */
        border-radius: 50%; /* يجعل الزر دائريًا */
        background-color: #6c757d; /* لون الخلفية */
        color: white; /* لون السهم */
        border: none; /* إزالة الحدود */
        font-size: 24px; /* حجم السهم */
        font-weight: bold; /* سمك السهم */
        box-shadow: 0 4px 10px rgba(0,0,0,0.3); /* ظل خفيف */
        cursor: pointer; /* مؤشر اليد عند المرور */
        transition: 0.3s; /* حركة ناعمة عند التفاعل */
        z-index: 9999; /* يظهر فوق كل العناصر */
    }

        /* تأثير عند تمرير الماوس */
        .floating-back-btn:hover {
            background-color: #5a6268; /* تغيير اللون */
            transform: scale(1.1); /* تكبير بسيط */
        }


    /* تنسيق عام للأزرار */
    .styled-btn {
        padding: 10px 25px;
        font-size: 16px;
        border-radius: 8px;
        font-weight: 600;
        border: none;
        transition: 0.3s;
        box-shadow: 0 3px 8px rgba(0,0,0,0.15);
    }

    /* زر الرجوع */
    #backButton {
        background-color: #6c757d;
        color: white;
    }

        #backButton:hover {
            background-color: #5a6268;
            transform: translateY(-2px);
        }

    /* زر التالي */
    #nextButton {
        background-color: #28a745;
        color: white;
    }

        #nextButton:hover {
            background-color: #218838;
            transform: translateY(-2px);
        }

    /* مسافة بسيطة بين الزرين */
    .btn-container {
        display: flex;
        gap: 10px;
        margin-bottom: 15px;
    }
</style>

<div class="container mt-4">
    <!-- حاوية Bootstrap لتهيئة الصفحة -->
    <!-- نموذج الإدخال والتعديل -->
    <div class="card mb-4">
        <!-- بطاقة Bootstrap -->
        <div class="card-header">إضافة / تعديل معلومات القرار</div>   <!-- عنوان البطاقة -->

        <div class="card-body">

            <input type="hidden" id="decisionsToBeMadeId" />
            <!-- يخزن رقم السجل عند التعديل -->

            <div class="mb-3">
                <label class="form-label">وصف القرار المطلوب اتخاذه</label>
                <textarea id="decisionsToBeMadeDesc" class="form-control"></textarea>
                <!-- حقل نصي طويل لوصف القرار -->
                <div id="descError" class="text-danger"></div>
                <!-- مكان لعرض خطأ التحقق -->
            </div>

            <div class="mb-3">
                <label class="form-label">رقم القرار</label>
                <input type="number" id="decisionsToBeMadeNo" class="form-control" />
                <!-- رقم القرار -->
            </div>

            <div class="mb-3">
                <label class="form-label">توصيات اللجنة</label>
                <textarea id="committeeRecommendations" class="form-control"></textarea>
                <!-- توصيات اللجنة -->
            </div>

            <div class="mb-3">
                <label class="form-label">رقم الاجتماع</label>
                <input type="number" id="meetingId" class="form-control" />
                <!-- رقم الاجتماع المرتبط بالقرار -->
            </div>

            <button class="btn btn-primary" id="saveDecision">حفظ</button>
            <!-- زر الحفظ (إضافة أو تعديل) -->

        </div>
    </div>
  
    <div class="btn-container">
        <button id="backButton" class="styled-btn">
            رجوع
        </button>

        <button id="nextButton" class="styled-btn" style="display:none;">
            التالي
        </button>
    </div>



    <!-- جدول عرض البيانات -->
    <div id="DecisionsTable"></div>



</div>

@section Scripts {

    <!-- سكربت JavaScript -->
    <script>

        // تحميل البيانات عند فتح الصفحة
        loadDecisions();

        $("#backButton").click(function () {
            window.history.back();
        });
        //$("#backFloatingBtn").click(function () {
        //    window.history.back();   // الرجوع للشاشة السابقة
        //});


        function loadDecisions() {
            $.get("/Home/GetAllDecisions", function (res) {

                $("#DecisionsTable").empty(); // تفريغ البطاقات

                if (res.length === 0) {
                    // لا توجد سجلات → إخفاء زر التالي
                    $("#nextButton").hide();
                    return;
                }

                // توجد سجلات → إظهار زر التالي
                $("#nextButton").show();

                res.forEach(d => {

                    $("#DecisionsTable").append(`
                   <div class="card mb-3 shadow-sm decision-card">


                        <div class="card-body">

                            <div class="row mb-2">
                                <div class="col-4 label-title text-end">
الوصف:</div>
                                <div class="col-8">${d.decisionsToBeMadeDesc}</div>
                            </div>

                            <div class="row mb-2">
                                <div class="col-4 label-title text-end">
رقم القرار:</div>
                                <div class="col-8">${d.decisionsToBeMadeNo}</div>
                            </div>

                            <div class="row mb-2">
                                <div class="col-4 label-title text-end">
توصيات اللجنة:</div>
                                <div class="col-8">${d.committeeRecommendations ?? ""}</div>
                            </div>

                            <div class="row mb-3">
                                <div class="col-4 label-title text-end">
رقم الاجتماع:</div>
                                <div class="col-8">${d.meetingId}</div>
                            </div>

                            <div class="text-start">
                                <button class="btn btn-warning btn-sm edit-btn" data-id="${d.decisionsToBeMadeId}">تعديل</button>
                                <button class="btn btn-danger btn-sm delete-btn" data-id="${d.decisionsToBeMadeId}">حذف</button>
                            </div>

                        </div>
                    </div>
                `);

                });
            });
        }

        $("#nextButton").click(function () {
            window.location.href = "/Home/Justification"; // غيّر المسار حسب رغبتك
        });




        // زر الحفظ (إضافة أو تعديل)
        $("#saveDecision").click(function () {

            $("#descError").text("");
            // تنظيف الأخطاء

            let model = {
                // إنشاء كائن يحتوي بيانات النموذج
                decisionsToBeMadeId: $("#decisionsToBeMadeId").val(),
                decisionsToBeMadeDesc: $("#decisionsToBeMadeDesc").val().trim(),
                decisionsToBeMadeNo: $("#decisionsToBeMadeNo").val(),
                committeeRecommendations: $("#committeeRecommendations").val(),
                meetingId: $("#meetingId").val()
            };

            if (model.decisionsToBeMadeDesc === "") {
                // التحقق من أن الوصف غير فارغ
                $("#descError").text("الرجاء إدخال وصف القرار");
                return;
            }

            $.ajax({
                url: "/Home/SaveDecision",
                type: "POST",
                contentType: "application/json",
                data: JSON.stringify(model),
                // إرسال البيانات كـ JSON

                success: function (res) {
                    if (res.success) {
                        loadDecisions();
                        // إعادة تحميل الجدول

                        // تفريغ الحقول
                        $("#decisionsToBeMadeId").val("");
                        $("#decisionsToBeMadeDesc").val("");
                        $("#decisionsToBeMadeNo").val("");
                        $("#committeeRecommendations").val("");
                        $("#meetingId").val("");
                    } else {
                        alert(res.message);
                    }
                }
            });
        });

        // زر التعديل
        $(document).on("click", ".edit-btn", function () {

            let id = $(this).data("id");
            // الحصول على رقم السجل

            $.get("/Home/GetDecisionById", { id: id }, function (d) {
                // جلب بيانات السجل من السيرفر

                $("#decisionsToBeMadeId").val(d.decisionsToBeMadeId);
                $("#decisionsToBeMadeDesc").val(d.decisionsToBeMadeDesc);
                $("#decisionsToBeMadeNo").val(d.decisionsToBeMadeNo);
                $("#committeeRecommendations").val(d.committeeRecommendations);
                $("#meetingId").val(d.meetingId);
            });
        });

        // زر الحذف
        $(document).on("click", ".delete-btn", function () {

            let id = $(this).data("id");
            // رقم السجل المطلوب حذفه

            if (!confirm("هل أنت متأكد من حذف هذا السجل؟")) return;

            $.post("/Home/DeleteDecision", { id: id }, function (res) {
                if (res.success) {
                    loadDecisions();
                    // إعادة تحميل الجدول بعد الحذف
                } else {
                    alert(res.message);
                }
            });
        });

    </script>

}
----------------------------
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Mvc;
using Webmeetingapp.Models;
using Webmeetingapp.ViewModel;

namespace Webmeetingapp.Controllers
{
    public class HomeController : Controller
    {
        private MeetingDbEntities db = new MeetingDbEntities();

        public ActionResult Index(int? meetingId)
        {
            if (meetingId == null) meetingId = 122; // قيمة افتراضية للاختبار

            var model = db.meetingOption
                .Where(x => x.meetingOptionMeetingId == meetingId)
                .Select(x => new OptionViewModel
                {
                    OptionId = x.meetingOptionId,
                    OptionName = x.meetingOptionTitle,
                    ProsCount = db.availableOptionOfMeeting
                        .Count(p => p.meetingOptionId == x.meetingOptionId && p.availableOptionType == "1"),
                    ConsCount = db.availableOptionOfMeeting
                        .Count(p => p.meetingOptionId == x.meetingOptionId && p.availableOptionType == "2")
                })
                .ToList();


            return View(model);
        }
        public ActionResult GetOptionById(int id)
        {
            try
            {
                // جلب بيانات الخيار الأساسي
                var option = db.meetingOption
                    .FirstOrDefault(o => o.meetingOptionId == id);

                if (option == null)
                {
                    return Json(new
                    {
                        success = false,
                        message = "الخيار غير موجود في قاعدة البيانات."
                    }, JsonRequestBehavior.AllowGet);
                }

                // جلب الإيجابيات (type = 1)
                var pros = db.availableOptionOfMeeting
                    .Where(a => a.meetingOptionId == id && a.availableOptionType == "1")
                    .Select(a => a.availableOptionDesc)
                    .ToList();

                // جلب السلبيات (type = 2)
                var cons = db.availableOptionOfMeeting
                    .Where(a => a.meetingOptionId == id && a.availableOptionType == "2")
                    .Select(a => a.availableOptionDesc)
                    .ToList();

                // تجهيز النموذج للإرجاع
                var model = new
                {
                    optionId = option.meetingOptionId,
                    optionName = option.meetingOptionTitle,
                    pros = pros ?? new List<string>(),
                    cons = cons ?? new List<string>()
                };

                return Json(new
                {
                    success = true,
                    data = model
                }, JsonRequestBehavior.AllowGet);
            }
            catch (Exception ex)
            {
                return Json(new
                {
                    success = false,
                    message = "حدث خطأ أثناء جلب البيانات.",
                    error = ex.Message
                }, JsonRequestBehavior.AllowGet);
            }
        }

        // ---------------------------------------------------------
        // 1) عرض الصفحة + تحميل جميع الخيارات
        // ---------------------------------------------------------
        public ActionResult MeetingOptions(int meetingId)
        {
            // جلب جميع الخيارات الخاصة بالاجتماع وتحويلها إلى ViewModel
            var model = db.meetingOption
                .Where(x => x.meetingOptionMeetingId == meetingId)   // فلترة حسب رقم الاجتماع
                .Select(x => new OptionViewModel
                {
                    OptionId = x.meetingOptionId,                    // رقم الخيار
                    OptionName = x.meetingOptionTitle,               // اسم الخيار

                    // حساب عدد الإيجابيات
                    ProsCount = db.availableOptionOfMeeting
                        .Count(p => p.meetingOptionId == x.meetingOptionId && p.availableOptionType == "1"),

                    // حساب عدد السلبيات
                    ConsCount = db.availableOptionOfMeeting
                        .Count(p => p.meetingOptionId == x.meetingOptionId && p.availableOptionType == "2")
                })
                .ToList();

            return View(model);                                      // إرسال البيانات للواجهة
        }

        
        

        [HttpGet]
        public ActionResult GetOptionDetails(int id)
        {
            try
            {
                var option = db.meetingOption
                    .FirstOrDefault(o => o.meetingOptionId == id);

                if (option == null)
                {
                    return Json(new
                    {
                        success = false,
                        message = "الخيار غير موجود في قاعدة البيانات."
                    }, JsonRequestBehavior.AllowGet);
                }

                var pros = db.availableOptionOfMeeting
                    .Where(a => a.meetingOptionId == id && a.availableOptionType == "1")
                    .Select(a => a.availableOptionDesc)
                    .ToList();

                var cons = db.availableOptionOfMeeting
                    .Where(a => a.meetingOptionId == id && a.availableOptionType == "2")
                    .Select(a => a.availableOptionDesc)
                    .ToList();

                var model = new
                {
                    optionId = option.meetingOptionId,
                    optionName = option.meetingOptionTitle,
                    pros = pros ?? new List<string>(),
                    cons = cons ?? new List<string>()
                };

                return Json(new { success = true, data = model }, JsonRequestBehavior.AllowGet);
            }
            catch (Exception ex)
            {
                return Json(new
                {
                    success = false,
                    message = "حدث خطأ أثناء جلب البيانات.",
                    error = ex.Message
                }, JsonRequestBehavior.AllowGet);
            }
        }

       

        [HttpPost]
        public ActionResult SaveOrUpdateOption(OptionEditViewModel model)
        {
            try
            {
                if (model.MeetingId == 0)
                    return Json(new { success = false, message = "معرف غير صالح." });

                if (string.IsNullOrWhiteSpace(model.OptionName))
                    return Json(new { success = false, message = "اسم الخيار مطلوب." });

                model.Pros = model.Pros ?? new List<string>();
                model.Cons = model.Cons ?? new List<string>();

                if (model.Pros.Any(p => string.IsNullOrWhiteSpace(p)))
                    return Json(new { success = false, message = "لا يمكن ترك الإيجابيات فارغة." });

                if (model.Cons.Any(c => string.IsNullOrWhiteSpace(c)))
                    return Json(new { success = false, message = "لا يمكن ترك السلبيات فارغة." });

                meetingOption option;

                using (var transaction = db.Database.BeginTransaction())
                {
                    if (model.Id == null)
                    {
                        option = new meetingOption
                        {
                            meetingOptionTitle = model.OptionName,
                            meetingOptionMeetingId = model.MeetingId,
                            meetingOptionAddedOn = DateTime.Now
                        };

                        db.meetingOption.Add(option);
                        db.SaveChanges();
                    }
                    else
                    {
                        option = db.meetingOption.Find(model.Id);

                        if (option == null)
                            return Json(new { success = false, message = "الخيار غير موجود." });

                        option.meetingOptionTitle = model.OptionName;

                        var old = db.availableOptionOfMeeting
                            .Where(a => a.meetingOptionId == model.Id);

                        db.availableOptionOfMeeting.RemoveRange(old);
                        db.SaveChanges();
                    }

                    foreach (var p in model.Pros)
                    {
                        db.availableOptionOfMeeting.Add(new availableOptionOfMeeting
                        {
                            meetingOptionId = option.meetingOptionId,
                            availableOptionType = "1",
                            availableOptionDesc = p,
                            availableOptionAddedOn = DateTime.Now
                        });
                    }

                    foreach (var c in model.Cons)
                    {
                        db.availableOptionOfMeeting.Add(new availableOptionOfMeeting
                        {
                            meetingOptionId = option.meetingOptionId,
                            availableOptionType = "2",
                            availableOptionDesc = c,
                            availableOptionAddedOn = DateTime.Now
                        });
                    }

                    db.SaveChanges();
                    transaction.Commit();
                }

                return Json(new { success = true, id = option.meetingOptionId });
            }
            catch (Exception ex)
            {
                return Json(new { success = false, message = "حدث خطأ أثناء الحفظ.", error = ex.Message });
            }
        }



        // ---------------------------------------------------------
        // 4) حذف خيار كامل
        // ---------------------------------------------------------
        [HttpPost]
        public ActionResult DeleteOption(int id)
        {
            try
            {
                var option = db.meetingOption.FirstOrDefault(o => o.meetingOptionId == id);

                if (option == null)
                {
                    return Json(new { success = false, message = "الخيار غير موجود." });
                }

                // حذف الإيجابيات والسلبيات المرتبطة
                var items = db.availableOptionOfMeeting
                    .Where(a => a.meetingOptionId == id)
                    .ToList();

                db.availableOptionOfMeeting.RemoveRange(items);

                // حذف الخيار نفسه
                db.meetingOption.Remove(option);

                db.SaveChanges();

                return Json(new { success = true });
            }
            catch (Exception ex)
            {
                return Json(new { success = false, message = "حدث خطأ أثناء الحذف.", error = ex.Message });
            }
        }
        //---------------------------------------------------------
        //بداية اضافة معلومات القرار
        //--------------------------------------------------------- 

        public ActionResult DecisionsToBeMade()
        {
            return View();
        }

        public ActionResult GetAllDecisions()
        {
            var list = db.decisionsToBeMade
                .Select(d => new {
                    d.decisionsToBeMadeId,
                    d.decisionsToBeMadeDesc,
                    d.decisionsToBeMadeNo,
                    d.committeeRecommendations,
                    d.meetingId
                }).ToList();

            return Json(list, JsonRequestBehavior.AllowGet);
        }


        public ActionResult GetDecisionById(int id)
        {
            var d = db.decisionsToBeMade.FirstOrDefault(x => x.decisionsToBeMadeId == id);
            return Json(d, JsonRequestBehavior.AllowGet);
        }
        [HttpPost]
        public ActionResult SaveDecision(decisionsToBeMade model)
        {
            try
            {
                if (model.decisionsToBeMadeId == 0)
                {
                    db.decisionsToBeMade.Add(model);
                }
                else
                {
                    var old = db.decisionsToBeMade.Find(model.decisionsToBeMadeId);

                    old.decisionsToBeMadeDesc = model.decisionsToBeMadeDesc;
                    old.decisionsToBeMadeNo = model.decisionsToBeMadeNo;
                    old.committeeRecommendations = model.committeeRecommendations;
                    old.meetingId = model.meetingId;
                }

                db.SaveChanges();

                return Json(new { success = true });
            }
            catch (Exception ex)
            {
                return Json(new { success = false, message = ex.Message });
            }
        }

        [HttpPost]
        public ActionResult SaveDecision1(decisionsToBeMade model)
        {
            try
            {
                if (model.decisionsToBeMadeId == 0)
                {
                    db.decisionsToBeMade.Add(model);
                }
                else
                {
                    var old = db.decisionsToBeMade.Find(model.decisionsToBeMadeId);

                    old.decisionsToBeMadeDesc = model.decisionsToBeMadeDesc;
                    old.decisionsToBeMadeNo = model.decisionsToBeMadeNo;
                    old.committeeRecommendations = model.committeeRecommendations;
                    old.meetingId = model.meetingId;
                }

                db.SaveChanges();
                return Json(new { success = true });
            }
            catch (Exception ex)
            {
                return Json(new { success = false, message = ex.Message });
            }
        }

        [HttpPost]
        public ActionResult DeleteDecision(int id)
        {
            try
            {
                var d = db.decisionsToBeMade.Find(id);
                if (d == null)
                    return Json(new { success = false, message = "السجل غير موجود" });

                db.decisionsToBeMade.Remove(d);
                db.SaveChanges();

                return Json(new { success = true });
            }
            catch (Exception ex)
            {
                return Json(new { success = false, message = ex.Message });
            }
        }


        //---------------------------------------------------------
        //بداية اضافة المبررات
        //--------------------------------------------------------- 
        
             public ActionResult Justification()
        {
            return View();
        }
        // 1) جلب جميع المبررات
        public ActionResult GetAllJustifications()
        {
            var list = db.Justification
                .Select(j => new
                {
                    j.JustificationsId,
                    j.JustificationsDesc,
                    j.meetingId
                })
                .ToList();

            return Json(list, JsonRequestBehavior.AllowGet);
        }

        // 2) جلب مبرر واحد للتعديل
        public ActionResult GetJustificationById(int id)
        {
            var j = db.Justification
                .Where(x => x.JustificationsId == id)
                .Select(x => new
                {
                    x.JustificationsId,
                    x.JustificationsDesc,
                    x.meetingId
                })
                .FirstOrDefault();

            return Json(j, JsonRequestBehavior.AllowGet);
        }

        // 3) إضافة أو تعديل مبرر
        [HttpPost]
        public ActionResult SaveJustification(Justification model)
        {
            try
            {
                if (model.JustificationsId == 0)
                {
                    // إضافة جديدة
                    db.Justification.Add(model);
                }
                else
                {
                    // تعديل
                    var old = db.Justification.Find(model.JustificationsId);

                    old.JustificationsDesc = model.JustificationsDesc;
                    old.meetingId = model.meetingId;
                }

                db.SaveChanges();

                return Json(new { success = true });
            }
            catch (Exception ex)
            {
                return Json(new { success = false, message = ex.Message });
            }
        }

        // 4) حفظ وإرسال (بدون منع التعديل)
        [HttpPost]
        public ActionResult SubmitJustificationFinal(Justification model)
        {
            try
            {
                var item = db.Justification.Find(model.JustificationsId);

                item.JustificationsDesc = model.JustificationsDesc;
                item.meetingId = model.meetingId;

                db.SaveChanges();

                return Json(new { success = true });
            }
            catch (Exception ex)
            {
                return Json(new { success = false, message = ex.Message });
            }
        }

        // 5) حذف المبرر
        [HttpPost]
        public ActionResult DeleteJustification(int id)
        {
            try
            {
                var item = db.Justification.Find(id);

                if (item == null)
                    return Json(new { success = false, message = "المبرر غير موجود" });

                db.Justification.Remove(item);
                db.SaveChanges();

                return Json(new { success = true });
            }
            catch (Exception ex)
            {
                return Json(new { success = false, message = ex.Message });
            }
        }
    }


}

 
