.factory('confirmService', ['$rootScope', '$timeout', '$q', function ($rootScope, $timeout, $q) {
  $rootScope.confirmMessage = false;
  var messageService = {
    newConfirm: function (msg, type) {
      return $rootScope.confirmMessage = ConfirmMessage(msg, type);
    }
  }

  /**
   * [ConfirmMessage constructor function creates a confirm object]
   * @param {[string]} msg  [message shown to user]
   * @param {[string]} type [bootrap panel class type e.g., 'danger']
   */
  function ConfirmMessage(msg, type) {
    type = type || 'default';
    var confirm;
    var promise = $q(function(resolve) {
      confirm = function (res) {
        resolve(res);
      }
    });
    return {
      promise: promise,
      msg: msg,
      type: type,
      continue: function () {
        $rootScope.confirmMessage = false;
        confirm(true);
      },
      cancel: function () {
        $rootScope.confirmMessage = false;
        confirm(false);
      }
    }
  }

  return messageService;
}])

// controller

$scope.delete = function (host) {
    var q = confirmService.newConfirm('testing', 'danger').promise;
    q.then(function(res) {
      if (res) {
        destroy(host)
      }
    });
  }
  
  // html
  <div ng-show="confirmMessage" class="panel panel-{{confirmMessage.type}} panel-confirm">
      <div class="panel-heading">{{confirmMessage.msg}}</div>
      <div class="panel-body">
        <button ng-click="confirmMessage.cancel()" class="btn btn-warning">Cancel</button>
        <button ng-click="confirmMessage.continue()" class="btn btn-success">Continue</button>
      </div>
    </div>
    
    //css
    
.panel-confirm {
  width: 33.33%;
  position: fixed;
  left: 33.33%;
  z-index: 10;
  top: 33.33%;
  box-shadow: 4px 3px 23px -6px rgba(0,0,0,0.75);
}
  

