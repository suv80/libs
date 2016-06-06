---
title: 重写js中的window.alert函数
tags: [javascript]
date: 2015/01/24
---

使用了bootstrap框架，所以要引入bootstrap框架。

```
/**
 * msg string 消息内容
 * title string 对话框标题
 * callback function 返回函数。在隐藏并且CSS动画结束后触发
 **/
window.alert = function (msg, title, callback) {
    if (!title) {
        title = '对话框';
    }
    var dialogHTML = '<div id="selfAlert" class="modal fade">';
    dialogHTML += '<div class="modal-dialog">';
    dialogHTML += '<div class="modal-content">';
    dialogHTML += '<div class="modal-header">';
    dialogHTML += '<button type="button" class="close" data-dismiss="modal" aria-label="Close">';
    dialogHTML += '<span aria-hidden="true">&times;</span>';
    dialogHTML += '</button>';
    dialogHTML += '<h4 class="modal-title">' + title + '</h4>';
    dialogHTML += '</div>';
    dialogHTML += '<div class="modal-body">';
    dialogHTML += msg;
    dialogHTML += '</div>';
    dialogHTML += '<div class="modal-footer">';
    dialogHTML += '<button type="button" class="btn btn-primary" data-dismiss="modal">确定</button>';
    dialogHTML += '</div>';
    dialogHTML += '</div>';
    dialogHTML += '</div>';
    dialogHTML += '</div>';

    if ($('#selfAlert').length <= 0) {
        $('body').append(dialogHTML);
    }

    $('#selfAlert').on('hidden.bs.modal', function () {
        $('#selfAlert').remove();
        if (typeof callback == 'function') {
            callback();
        }
    }).modal('show');
}
```
