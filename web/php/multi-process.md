# PHP Multi Process

```php
class ProcessUtil
{
    public static function spawn($callback, $processCount = 3)
    {
        for ($index = 0; $index < $processCount; $index++) {
            $pid = pcntl_fork();
            if ($pid == 0) {
                $process = ['index' => $index, 'count' => $processCount];
                call_user_func_array($callback, [$process]);
                exit(0);
            }
        }

        $exitedProcessCount = 0;
        while ($exitedProcessCount < $processCount) {
            $status = -1;
            // 这里应该阻塞，之前踩过一个坑 `$option = WNOHANG` 造成长时间 while 循环占用大量内存(http://www.php.net/manual/en/function.pcntl-wait.php)
            $pid = pcntl_wait($status);
            if ($pid > 0) {
                echo("process {$pid} exit\n");
                $exitedProcessCount++;
            }
        }
    }
}
```
