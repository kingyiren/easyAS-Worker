# Easy AS Workers for Adobe Air AS3 projects
This AS3 library lets you run AS workers in your AdobeAir projects, desktop, browser, Android and iOS without you having to deal with complicated classic ```flash.system.Worker;``` class! [**Check here for more setup details**](http://www.myflashlabs.com/developer-friendly-as-worker-api/)

# USAGE
First create your worker class where you need to run your heavy code algurithm. take ```Worker1``` class as an example.
```actionscript
package workers
{
	import com.myflashlabs.utils.worker.WorkerBase;
	
	/**
	 * ...
	 * @author MyFlashLab Team - 1/28/2016 11:00 PM
	 */
	public class Worker1 extends WorkerBase
	{
		
		public function Worker1()
		{
		
		}
		
		// these methods must be public because they are called from the main thread.
		public function forLoop($myParam:int):void
		{
			var thisCommand:Function = arguments.callee;
			
			for (var i:int = 0; i < $myParam; i++)
			{
				// call this method to send progress to your delegate
				sendProgress(thisCommand, i);
			}
			
			// call this method as the final message from the worker. When this is called, you cannot send anymore "sendProgress"
			sendResult(thisCommand, $myParam);
		}
	
	}
}
```

Then, in your main project, all you need to do is to initialize ```com.myflashlabs.utils.worker.WorkerManager``` and run your worker whenever you wish.
```actionscript
package
{
	import flash.display.Sprite;
	import flash.events.Event;
	import flash.system.WorkerState;
	import flash.text.TextField;
	import flash.text.TextFieldAutoSize;
	import flash.text.TextFormat;
	
	import workers.Worker1;
	
	// WorkerManager which will be taking care of all complicated stuff about AS Workers for you
	import com.myflashlabs.utils.worker.WorkerManager;
	
	/**
	 * ...
	 * @author Hadi Tavakoli - 1/29/2016 1:17 AM
	 */
	public class Main extends Sprite
	{
		private var _myWorker:WorkerManager;
		private var _txt:TextField;
		
		public function Main()
		{
			_txt = new TextField();
			_txt.autoSize = TextFieldAutoSize.LEFT;
			_txt.defaultTextFormat = new TextFormat(null, 40);
			_txt.x = _txt.y = 50;
			this.addChild(_txt);
			
			// init the Manager and pass the class you want to use as your Worker
			_myWorker = new WorkerManager(workers.Worker1, loaderInfo.bytes, this);
			
			// listen to your worker state changes
			_myWorker.addEventListener(Event.WORKER_STATE, onWorkerState);
			
			// fire up the Worker
			_myWorker.start();
		}
		
		private function onWorkerState(e:Event):void
		{
			trace("worker state = " + _myWorker.state)
			
			// if the worker state is 'running', you can start communicating
			if (_myWorker.state == WorkerState.RUNNING)
			{
				// create your own commands in your worker class, Worker1, i.e "forLoop" in this sample and pass in as many parameters as you wish
				_myWorker.command("forLoop", onProgress, onResult, 10000);
			}
		}
		
		/**
		 * this function can have as many parameters as you wish. 
		 * this is just a contract between the worker class and this delegate.
		 * What you need to notice though, is that it must return void.
		 */
		private function onProgress($progress:Number):void
		{
			_txt.text = "$progress = " + $progress;
		}
		
		/**
		 * this function can have as many parameters as you wish. 
		 * this is just a contract between the worker class and this delegate.
		 * What you need to notice though, is that it must return void.
		 */
		private function onResult($result:Number):void
		{
			_txt.text = "$result = " + $result;
			
			// terminate the worker when you're done with it.
			_myWorker.terminate();
		}
	}
}
```