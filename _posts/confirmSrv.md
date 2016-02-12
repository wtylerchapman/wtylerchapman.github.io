.factory('confirmService', ['$rootScope', '$timeout', function ($rootScope, $timeout) {
  $rootScope.confirmMessage = false;
  var messageService = {
    newConfirm: function (msg) {
      var cMsg = ConfirmMessage(msg);
      $rootScope.confirmMessage = cMsg;
    }
  }

  function ConfirmMessage(msg) {
    return {
      msg: msg,
      continue: function () {
        $rootScope.confirmMessage = false;
        return true;
      },
      cancel: function () {
        $rootScope.confirmMessage = false;
        return false;
      }
    }
  }

  return messageService;
}])
