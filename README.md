# easyui.datagrid.cellmove
make easyui-datagrid cell cound drag.

## create methods "columnMoving" extends $.fn.datagrid.methods 
```javascript
function columnMoving(jq, opts){
    return jq.each(function(){
            var target = this;
            var cells = $(this).datagrid('getPanel').find('div.datagrid-header td[field]');
            var options = $.extend({
                revert:true,
                cursor:'pointer',
                edge:5,
                proxy: function(source){
                    var p = $('<div class="tree-node-proxy tree-dnd-no" style="position:absolute;border:1px solid #ff0000"/>');
                    p.appendTo('body')
                    p.html($(source).text());
                    p.hide();
                    return p;
                },
                onBeforeDrag:function(e){
                    e.data.startLeft = $(this).offset().left;
                    e.data.startTop = $(this).offset().top;
                },
                onStartDrag: function(){
                    $(this).draggable('proxy').css({
                        left:-10000,
                        top:-10000
                    });
                },
                onDrag:function(e){
                    $(this).draggable('proxy').show().css({
                        left:e.pageX+15,
                        top:e.pageY+15
                    });
                    return false;
                }
            }, opts);
            cells.draggable(options).droppable({
                accept:'td[field]',
                onDragOver:function(e,source){
                    $(source).draggable('proxy').removeClass('tree-dnd-no').addClass('tree-dnd-yes');
                    $(this).css('border-left','1px solid #ff0000');
                },
                onDragLeave:function(e,source){
                    $(source).draggable('proxy').removeClass('tree-dnd-yes').addClass('tree-dnd-no');
                    $(this).css('border-left',0);
                },
                onDrop:function(e,source){
                    $(this).css('border-left',0);
                    var fromField = $(source).attr('field');
                    var toField = $(this).attr('field');
                    setTimeout(function(){
                        if(swapField(fromField,toField)){
                            $(target).datagrid();
                            $(target).datagrid('columnMoving', opts);
                            if(opts.onStopDrop){
                                opts.onStopDrop();
                            }
                        }
                    },0);
                }
            });
        });
    }
}

//判断
function fieldsInColumns(from, to, columns){
    var index = -1;
    for(var i=0; i<columns.length; i++){
        var _from = false, _to = false;
        var column = columns[i];
        for(var j=0; j<column.length; j++){
            if(column[j].field == from){
                _from = true;
                if(_to) break;
            }else if(column[j].field == to){
                _to = true;
                if(_from) break;
            }
        }
        if(_from && _to){
            index = i;
            break;
        }
    }
    return index;
}                        

// swap Field to another location
function swapField(from,to){
    var columns = $(target).datagrid('options').columns;
    var frozenColumns = $(target).datagrid('options').frozenColumns;
    var newColumns = [];

    //交叉拖拽与跨行（合并单元格）不被允许，此处判断两列在不在一个列表中
    var index = fieldsInColumns(from, to, columns);
    if(index != -1){
        newColumns = columns;
    }else{
        index = fieldsInColumns(from, to, frozenColumns);
        if(index != -1){
            newColumns = frozenColumns;
        }else{
            return false;
        }
    }
    var cc = newColumns[index];
    var fromtemp;
    var totemp;
    var fromindex = 0;
    var toindex = 0;
    for(var i=0; i<cc.length; i++){
        if (cc[i].field == from){
            fromindex = i;
            fromtemp = cc[i];
        }
        if(cc[i].field == to){
            toindex = i;
            totemp = cc[i];
        }
    }

    //如果有parent字段，不同的parent之间不允许移动
    if(!cc[fromindex].parent && !cc[toindex].parent){
        cc.splice(fromindex,1,totemp);
        cc.splice(toindex,1,fromtemp);
    }else{ 
        if(cc[fromindex].parent && cc[toindex].parent && (cc[fromindex].parent == cc[toindex].parent)){
            cc.splice(fromindex,1,totemp);
            cc.splice(toindex,1,fromtemp);
        }else{
            return false;
        }
    }

    //处理所拖动行的下面所有行
    for(var i=index+1; i<newColumns.length; i++){
        var dd = newColumns[i];
        var _fromend = 0;
        var _toend = 0;
        var _fromtemp = [], _totemp = [];
        for(var j=0; j<dd.length; j++){
            if (dd[j].parent == from){
                _fromend = j;
                _fromtemp.push(dd[j]);
            }
            if(dd[j].parent == to){
                _toend = j;
                _totemp.push(dd[j]);
            }
        }
        var _fromlen = _fromtemp.length,
            _tolen = _totemp.length;
        if(_fromlen > 0 && _tolen > 0){
            var _fromstart = _fromend - _fromlen + 1;
            var _tostart = _toend - _tolen + 1;
            if(_tostart > _fromstart){
                dd = dd.slice(0, _fromstart).concat(_totemp, dd.slice(_fromend+1, _tostart), _fromtemp, dd.slice(_toend, dd.length-1));
            }else{
                dd = dd.slice(0, _tostart).concat(_fromtemp, dd.slice(_toend+1, _fromstart), _totemp, dd.slice(_fromend, dd.length-1));
            }
        }
        newColumns[i] = dd;
    }
    return true;
}

$.extend($.fn.datagrid.methods,{
    columnMoving: columnMoving
});
```
